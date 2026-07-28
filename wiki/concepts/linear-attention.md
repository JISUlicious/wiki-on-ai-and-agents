---
title: Linear Attention
type: concept
created: 2026-07-28
updated: 2026-07-28
sources:
  - kimi-linear-kimi-team-2025.md
  - flash-kda-moonshot-2026.md
  - a-hippocampus-for-linear-attention-cui-2026.md
builds-on:
  - [[attention-mechanism]]
status: complete
importance: high
---

# Linear Attention

**Linear attention** replaces softmax attention's growing [[kv-cache|KV cache]] with a **fixed-size recurrent state**, trading O(N²) compute and O(N) memory for **O(N) compute and O(1) memory** in sequence length. Instead of attending over all previous tokens, the model carries a state matrix `S` updated token by token.

The whole design space follows from one consequence: **a fixed-size state has finite capacity, so it must overwrite.** Everything interesting in modern linear attention is a strategy for deciding *what* to forget — or for arranging not to have to.

## Why it matters for prefill

Linear attention is the structural answer to the quadratic term. Where [[flash-attention]] computes exact quadratic attention with far less memory traffic and [[minimax-sparse-attention-lai-2026|sparse attention]] skips blocks, linear attention **never incurs the quadratic cost at all**. In [[kimi-linear-kimi-team-2025|Kimi Linear]] this shows up as **2.3× prefill at 512k and 2.9× at 1M**, with the advantage only appearing past long context — *comparable to MLA at 4k–16k, diverging from 128k on*. Below ~100k the quadratic term simply isn't the bottleneck yet.

## The gating lineage

The line runs from plain linear attention → the **delta rule** (write a correction rather than blindly accumulate) → **Gated DeltaNet** (a forget gate controlling decay) → **KDA**.

**Kimi Delta Attention (KDA)** extends Gated DeltaNet with **channel-wise gating** — a per-feature-dimension decay `Diag(α_t)` rather than GDN's single scalar/head-wise gate:

```
S_t = (I − β_t k_t k_tᵀ) · Diag(α_t) · S_{t−1} + β_t k_t v_tᵀ
o_t = S_tᵀ q_t
```

Finer-grained gating means the state can forget *per dimension* instead of uniformly, using limited capacity better. The efficiency trick is that this is a **constrained** DPLR (diagonal-plus-low-rank) transition with both low-rank factors tied to `k` — which removes two secondary-chunking steps and ~3 matmuls, giving **~2× the kernel speed of general DPLR** up to 64k. Its optimized kernel is [[flash-kda-moonshot-2026|FlashKDA]].

## Nobody ships pure linear attention

The most important practical fact. [[kimi-linear-kimi-team-2025|Kimi Linear]] is a **hybrid: 3 KDA layers to 1 full-attention layer**, interleaved layerwise. The ablation is decisive and non-monotonic:

| Ratio (KDA:full) | Train / Val PPL |
|---|---|
| 1:1 | 9.29 / 5.66 |
| **3:1** | **9.23 / 5.65** |
| 7:1 | 9.23 / 5.70 |
| 15:1 | 9.34 / 5.82 |
| 0:1 (pure full attention) | 9.45 / 5.77 |

More linear layers is not better — 15:1 is worse than pure full attention on validation. The full-attention layers are **MLA**, and they exist precisely because **finite-state capacity caps exact recall and copying**. That is the standing limitation of the whole family.

Two responses to the same problem:
- **Hybridize** — keep some full-attention layers (Kimi Linear).
- **Add exact memory** — [[a-hippocampus-for-linear-attention-cui-2026|HOLA]] bolts a bounded exact KV cache (w=64/layer) onto a Gated DeltaNet state, evicting by the delta rule's own committed residual. Result: **−16.1% perplexity**, and 0.58 vs 0.14 on RULER S-NIAH-1 at **16× training length**.

## What you get, and what it costs

From Kimi Linear's matched 1.4T-token comparison (48B total / 3B active MoE):

| | Result |
|---|---|
| KV cache | **−75%** |
| Decode TPOT @1M | **6.3×** — but batch-scaled; **2.2× at batch 1** |
| Prefill | 2.3× @512k, 2.9× @1M |
| Quality vs full attention | MMLU **73.8 vs 71.6**, MMLU-Pro 51.0 vs 47.2, RULER-128k **84.3 vs 81.3** |
| Compute efficiency | ~1.16× (scaling law) |

> [!note] The headline decode number is batch-scaled
> The 6.3× TPOT at 1M comes largely from the smaller KV cache enabling **larger batches**, not from per-token work alone — at batch 1 it is 2.2×. The same indirection appears in KV-cache quantization (see [[quantization-performance]]): memory savings become throughput by way of concurrency.

An elegant side effect: with **NoPE** on all full-attention layers, KDA becomes the *sole position-aware operator* — the gated delta rule acts as a learnable, data-dependent multiplicative positional encoding. This also lets MLA collapse to pure MQA at inference and removes RoPE base tuning / YaRN entirely. The RoPE variant is worse on long context (RULER 78.8 vs 84.3) while matching on short.

## Honest limitations

Kimi Linear states them: finite-state capacity still caps exact recall (hence the hybrid); linear attention remains hampered by **hardware and inference-infrastructure maturity** — which is exactly the gap [[flash-kda-moonshot-2026|FlashKDA]] addresses; the 3:1 ratio is empirical rather than derived; and it **loses** on EvalPlus, MATH500, LiveBench, LongBench v2 and Frames. The authors also note sparse attention retrieves fine-grained history better, and combining the two is future work.

## Related

- [[attention-mechanism]] · [[state-space-model]] — the adjacent recurrent-state family (Mamba et al.).
- [[flash-attention]] — the exact-quadratic route; [[minimax-sparse-attention-lai-2026]] — the approximate-sparse route.
- [[kv-cache]] · [[long-context-llm]] · [[quantization-performance]] · [[moonshot-ai]]

## References

- [[kimi-linear-kimi-team-2025]] · [[flash-kda-moonshot-2026]] · [[a-hippocampus-for-linear-attention-cui-2026]]
