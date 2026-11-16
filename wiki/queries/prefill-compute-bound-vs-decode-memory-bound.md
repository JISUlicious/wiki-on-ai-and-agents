---
title: "Why is prefill compute-bound and token generation memory-bandwidth-bound?"
type: query
created: 2026-11-16
updated: 2026-11-16
sources:
  - llm-inference-roofline-yuan-2024.md
  - quantization-performance.md
  - distserve-zhong-2024.md
  - prefill-decode-aware-evaluation-ai-accelerators-usami-2026.md
  - hbm-is-not-all-you-need-wei-2026.md
status: complete
importance: high
---

# Why is prefill compute-bound and token generation memory-bandwidth-bound?

**Question (2026-11-16):** Why are prefill and token generation tied to different aspects of processing, compute and memory bandwidth?

Parts marked **(inference)** are back-of-envelope reasoning, not figures from a source.

## Short answer

Prefill and decode do the same math. What differs is how many times each weight gets used after it's fetched from memory. Prefill reuses it many times, so the processor is the bottleneck. Decode uses it once per token, so memory bandwidth is the bottleneck.

## The mechanism: work done per byte read

To compute anything, the GPU must first bring the weights from memory (HBM or DRAM) into the chip. The useful ratio is [[arithmetic-intensity]]:

`I = FLOPs / bytes moved`

Each chip has a **ridge point** (`peak FLOPS / peak bandwidth`). Above it, a step waits on arithmetic; below it, a step waits on memory. For an A6000 (155 TFLOPS, 768 GB/s) the ridge is about **200 operations per byte**.

**Prefill (prompt processing, `pp`)**
- All S prompt tokens are known up front, so the model processes them together.
- Each weight matrix multiplies a whole S-token matrix at once (a matrix–matrix product).
- One read of a weight serves S tokens, so **I ≈ S**, typically hundreds to thousands.
- The chip has the data it needs and spends its time on arithmetic. Prefill is **compute-bound**.

**Decode (token generation, `tg`)**
- Token t+1 depends on token t, so tokens come out one at a time.
- Each step multiplies one token's vector by every weight matrix (a matrix–vector product).
- Every weight is read once and used for about 2 FLOPs, so **I ≈ 1**.
- Each step also reads the entire [[kv-cache]] of past tokens, which grows with context.
- The arithmetic units mostly sit idle waiting for data. Decode is **memory-bandwidth-bound**.

The roofline survey [[llm-inference-roofline-yuan-2024]] measured this on Llama-2-7B:

| Phase | Step | Intensity (operations per byte) | Limited by |
|---|---|---|---|
| Prefill | q/k/v/o projections | 1,024 | compute |
| Prefill | FFN (gate/up/down) | 1,215 | compute |
| Prefill | QKᵀ attention matmul | 114 | **memory** |
| Decode | all projections | ~1 | memory |
| Decode | attention | 0.99 | memory |

The two phases of the same model differ by about **1000×**. Decode sits about 200× below the ridge.

## What each phase's speed depends on (inference)

- **Decode speed ≈ memory bandwidth ÷ bytes read per token.** A 7B model in FP16 is about 14 GB. At 800 GB/s that caps decode near 57 tokens/s, however fast the arithmetic is.
- **Prefill speed ≈ peak FLOPS ÷ (2 × parameters) per token.** That is about 14 GFLOP per token, so 155 TFLOPS gives roughly 11,000 tokens/s. Prefill can be hundreds of times faster per token than decode on the same chip.

## Why this matters: each phase needs different optimizations

| Lever | Effect on decode | Effect on prefill | Why |
|---|---|---|---|
| 4-bit weight-only [[quantization]] | **2.3–2.9× faster** | **2–12% slower** | It cuts bytes, which decode is limited by. Prefill isn't limited by bytes, and unpacking the 4-bit weights adds arithmetic. Measured with [[llama-cpp|llama.cpp]] on [[apple-silicon]] ([[quantization-performance]]). |
| [[grouped-query-attention]] | win | no gain | Smaller KV cache means fewer bytes per decode step. |
| Batching many users' decodes | big win | little gain | One weight read now serves B tokens, so I ≈ B and decode moves toward the ridge. DistServe found that a single 512-token prompt already saturates an A100 on a 13B model. |
| [[speculative-decoding]] | win | none | Checking k drafted tokens in one pass costs about the same as one decode step, because the compute was idle anyway. |
| [[flash-attention]] | some gain | win | Even in prefill, standard attention is memory-bound (I = 114) because it writes the N×N score matrix to memory. FlashAttention keeps that matrix on the chip. |
| [[prefix-caching]], [[sparse-prefill]], [[prompt-compression]] | none | win | They skip or shrink compute. See [[methods-to-speed-up-prompt-processing]]. |

## How servers and hardware handle the split

- **Different deadlines.** Prefill is judged by time to first token (TTFT), decode by time per output token (TPOT).
- **Mixing hurts decode.** If one GPU runs both, a long prefill stretches every decode step batched with it.
  - [[distserve-zhong-2024|DistServe]] puts prefill and decode on separate GPUs and gets up to **7.4×** more requests within latency targets.
  - [[chunked-prefill]] instead slices prefill into pieces mixed with decodes. DistServe argues this trades TTFT for TPOT and makes KV-cache reloads grow quadratically with the number of chunks.
- **Hardware follows the same line.**
  - A 4K-token prefill on an A100 leaves **about 97% of HBM bandwidth idle**. [[hbm-is-not-all-you-need-wei-2026|HBM Is Not All You Need]] therefore proposes cheaper GDDR memory for prefill and HBM for decode.
  - [[groq]]'s SRAM chips have huge on-chip bandwidth. On Llama2-7B a GroqRack beat one A100 by about **4× at decode** and lost by up to **39× at prefill** ([[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]]). The paper blames the prefill loss on a binary that runs prefill one token at a time.

## Exceptions

- **Long context:** at decode, reading the KV cache can cost more than reading the weights, which is why [[kv-cache-quantization]] and [[kv-cache-eviction]] matter. At prefill, attention grows from about ¼ of prefill time at 10K tokens to over 90% past 300K ([[sparse-prefill]]).
- **MoE models:** each token reads only the active experts. A field measurement on an M4 Air found decode limited by fixed per-token overhead rather than bytes: going from 4-bit to 3-bit made the model 1.28× smaller but only 1.09× faster ([[quantization-performance]]).
- **CPU:** prefill speed depends more on how well-optimized each format's matrix-multiply code is than on bit width ([[quantization-performance]]).

## References

- [[llm-inference-roofline-yuan-2024]] · [[arithmetic-intensity]] · [[quantization-performance]]
- [[distserve-zhong-2024]] · [[chunked-prefill]] · [[hbm-is-not-all-you-need-wei-2026]]
- [[groq]] · [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]]
- [[methods-to-speed-up-prompt-processing]]
