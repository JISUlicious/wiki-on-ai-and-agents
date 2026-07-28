---
title: "Biggest Memory and Compute Consumers in LLM Training and Inference"
type: query
created: 2026-07-29
updated: 2026-07-29
sources:
  - zero-memory-optimization-rajbhandari-2019.md
  - reducing-activation-recomputation-korthikanti-2022.md
  - megatron-ptd-parallelism-narayanan-2021.md
  - llm-inference-roofline-yuan-2024.md
  - mixed-precision-training-micikevicius-2017.md
tags:
  - 2026
status: complete
importance: high
---

# Biggest Memory and Compute Consumers in LLM Training and Inference

A sub-step breakdown across four quadrants: **training × inference** by **memory × compute**.

The single most useful generalization up front: **memory and compute bottlenecks almost never sit in the same sub-step.** Training memory is dominated by things that aren't the model; inference compute is dominated by the FFN while inference *memory* increasingly isn't. Optimizing the wrong quadrant is the most common efficiency mistake.

## Summary

| | **Memory** | **Compute** |
|---|---|---|
| **Training** | optimizer states (12 of 16 bytes/param), then activations — weights are ~2/16 | FFN matmuls, ×3 for the backward pass |
| **Inference** | weights → KV cache as context grows | prefill FFN; decode is bandwidth-bound, not compute-bound |

## 1. Training — memory

**Model states**, per parameter, mixed-precision Adam ([[zero-memory-optimization-rajbhandari-2019|ZeRO]]):

| Component | Bytes |
|---|---|
| fp16 parameters | 2Ψ |
| fp16 gradients | 2Ψ |
| fp32 master + momentum + variance | **12Ψ** |
| **Total** | **16Ψ** |

**Optimizer states are 75% of model-state memory.** GPT-2 1.5B: **≥24 GB** of model states vs **3 GB** of fp16 parameters.

**Activations**, per transformer layer ([[reducing-activation-recomputation-korthikanti-2022|Korthikanti et al.]]):

$$sbh\left(34 + \frac{5as}{h}\right)$$

`34sbh` is linear in sequence length; **`5as²b` is quadratic**. For GPT-3, `5as/h = 80` against 34 for the entire rest of the layer — **attention is ~70% of activation memory**.

**Verdict: optimizer states below ~4k context, activations above it.** Weights are never the answer.

Corroboration from three directions: [[scalable-moe-training-megatron-core-yan-2026|Megatron Core]] cut DeepSeek-V3 from **199.5 GB → under 80 GB/GPU** using recompute + BF16 moments + CPU offload — all targeting activations and optimizer state, none touching weights. [[qlora-dettmers-2023|QLoRA]] fits **65B on one 48 GB GPU** mainly because a frozen base has no gradients or optimizer states. And [[mixed-precision-training|mixed precision]] *raises* weight memory 50% (6 B/param vs 4 B) yet still reduces the total — because activations were the larger term.

See [[training-memory-anatomy]].

## 2. Training — compute

Closed form from [[megatron-ptd-parallelism-narayanan-2021|PTD]]:

$$F = 96\,B\,s\,l\,h^2\left(1 + \frac{s}{6h} + \frac{V}{16lh}\right)$$

| Term | What it is | When it matters |
|---|---|---|
| `96Bslh²` | dense matmuls: attention projections + FFN | always dominant |
| `s/6h` | **attention scores** (quadratic) | only once s approaches 6h |
| `V/16lh` | vocabulary / unembedding | large-vocab models |

Ranked by sub-step: **FFN > attention projections > attention scores > norms/softmax**. Two multipliers: **backward ≈ 2× forward** (hence ~3× total, the familiar `6N` vs `2N` per token), and the quadratic term takes over at long context.

A caveat from [[flash-attention-2-dao-2023|FlashAttention-2]]: **non-matmul FLOPs are ~16× more expensive** than matmul on an A100 (312 vs 19.5 TFLOPs/s). Softmax and normalization are cheap in FLOP *count* but disproportionately expensive in *time*.

**Verdict: FFN matmuls, tripled by backprop.**

## 3. Inference — memory

Two consumers, and which dominates **flips with context length**:

| Model | KV per token (FP16) | At full context |
|---|---|---|
| Llama-3-8B (GQA-8) | 128 KiB | 16.0 GiB @ 128k |
| **Llama-2-7B (MHA)** | 512 KiB | **16.0 GiB @ 32k** |
| Llama-3-70B (GQA-8) | 320 KiB | 40.0 GiB @ 128k |

The Llama-2-7B row is the headline: **at 32k the KV cache exceeds the FP16 weights** (16.0 vs 12.55 GiB). The cache does **not** scale with parameter count — it scales with `layers × kv_heads × head_dim × tokens × batch`.

[[nemotron-3-puzzle-75b-a9b-nvidia-2026|Puzzle-75B]] shows both regimes at once: at 1M context on one H100, weights (~70 GB) + ~4 GB KV/request saturate 80 GB → **concurrency 1**; compressing weights to 44.5 GB fits **8 concurrent requests**.

**Verdict: weights at short context, KV cache at long context.** [[grouped-query-attention|GQA]] exists to push that crossover right.

## 4. Inference — compute

Per-sub-step arithmetic intensity, Llama-2-7B on A6000, ridge point ~200 OPs/byte ([[llm-inference-roofline-yuan-2024]]):

| Phase | Sub-step | Intensity | Bound |
|---|---|---|---|
| **Prefill** | gate/up/down_proj (FFN) | **1,215** | compute |
| **Prefill** | q/k/v/o_proj | **1,024** | compute |
| **Prefill** | qk_matmul | **114** | **memory** |
| **Decode** | all projections | **1** | memory |
| **Decode** | attention ops | **0.99** | memory |

Two sub-step observations: **FFN outranks attention projections even in prefill**, and **`qk_matmul` is memory-bound even during prefill** — which is exactly the gap [[flash-attention]] closes.

**Decode has no meaningful compute cost.** At intensity ~1 it sits ~200× below the ridge; its cost is *streaming weights*, not arithmetic.

**Verdict: prefill FFN. Decode is a bandwidth problem.** See [[quantization-performance]] and [[methods-to-speed-up-prompt-processing]].

## The levers, matched to the bottleneck

| Bottleneck | Levers |
|---|---|
| Training memory — optimizer | [[zero-redundancy-optimizer\|ZeRO]] (4×/8×/N_d×), [[8-bit-optimizers-dettmers-2021\|8-bit optimizers]] (4×), CPU offload |
| Training memory — activations | [[activation-recomputation]] (selective: 70% saved for 2.7% FLOPs), sequence parallelism, [[flash-attention]] |
| Training compute | [[distributed-training\|TP/PP/DP]], [[mixed-precision-training\|mixed precision]], [[mixture-of-experts\|MoE]] |
| Inference memory | [[grouped-query-attention\|GQA]], KV quantization, [[paged-attention]], weight [[quantization]] |
| Inference compute | [[prefix-caching]] (skip prefill), [[flash-attention]], sparse/[[linear-attention]], FP8 |

## Cross-cutting note

The same quadratic term — attention over sequence length — is the binding constraint in **three of the four quadrants**: training activation memory (`5as²b`), training compute (`s/6h`), and inference memory (KV cache). Only inference compute escapes it, and only because decode is bandwidth-bound instead. That is why [[flash-attention]], [[linear-attention]] and sparse attention show up as answers to superficially unrelated problems.

## References

- [[training-memory-anatomy]] · [[distributed-training]] · [[zero-redundancy-optimizer]] · [[activation-recomputation]] · [[mixed-precision-training]]
- [[quantization-performance]] · [[llm-inference-roofline-yuan-2024]] · [[methods-to-speed-up-prompt-processing]]
