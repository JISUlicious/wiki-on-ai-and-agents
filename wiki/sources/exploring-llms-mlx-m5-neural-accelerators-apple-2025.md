---
title: "Exploring LLMs with MLX and the Neural Accelerators in the M5 GPU"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
authors:
  - Apple Machine Learning Research (MLX team)
year: 2025
venue: "Apple Machine Learning Research blog, 2025-11-19"
status: complete
importance: high
tags:
  - 2025
---

# Exploring LLMs with MLX and the Neural Accelerators in the M5 GPU

**Source**: [machinelearning.apple.com/research/exploring-llms-mlx-m5](https://machinelearning.apple.com/research/exploring-llms-mlx-m5), published 2025-11-19. [[apple|Apple]]'s own benchmark of the M5's GPU Neural Accelerators through [[mlx]].

This is a vendor benchmark of the vendor's own hardware and framework, on one machine, with no third-party reproduction held in this wiki. It is nonetheless the most consequential prefill number on this platform, because it is the only lever that moves time-to-first-token by more than 2× without touching the model.

## What the Neural Accelerators are

Dedicated matrix-multiplication units exposed through Metal 4's **Tensor Operations (TensorOps)** and the Metal Performance Primitives framework. The post does not state their placement or count; Apple's marketing describes one per GPU core. MLX uses them automatically — no code change — on **macOS 26.2 or later**.

## Setup

MacBook Pro **M5, 24 GB** unified memory, against a similarly configured MacBook Pro M4. Prompt **4,096 tokens**, then **128 generated tokens**. Models are Qwen3 family, GPT-OSS, and FLUX for image generation.

## Measured speedups, M5 over M4

| Model | Precision | Time-to-first-token | Token generation |
|---|---|---|---|
| Qwen3-1.7B | BF16 | 3.57× | ~1.19–1.27× |
| Qwen3-8B | BF16 | 3.62× | ~1.19–1.27× |
| Qwen3-8B | 4-bit | 3.97× | ~1.19–1.27× |
| **Qwen3-14B** | 4-bit | **4.06×** | ~1.19–1.27× |
| GPT-OSS 20B | MXFP4 | 3.33× | ~1.19–1.27× |
| Qwen3-30B-A3B (MoE) | 4-bit | 3.52× | ~1.19–1.27× |

The post gives the generation range only in aggregate; the per-model TTFT figures are as listed.

## Why the two phases diverge so sharply

The split is the [[arithmetic-intensity]] argument in hardware form. Prefill is compute-bound — a 4,096-token prompt is a large GEMM — and the accelerators are matrix units, so it takes the full gain. Decode is memory-bandwidth-bound, and the M5's bandwidth is only modestly higher than the M4's, so token generation moves by the bandwidth ratio and nothing more. The 4-bit rows gain slightly *more* on TTFT than BF16, which is consistent with dequantization overhead being absorbed by the new units.

## What this means for the rest of the wiki's prefill advice

> [!note] Hardware first, then software
> Every software lever on [[methods-to-speed-up-prompt-processing]] — [[prefix-caching]], [[sparse-prefill]], [[speculative-prefill]], token pruning — is worth 1.1–2.3× on a cold prompt at the lengths a laptop sees. A generation change is worth 3.3–4.1× on the same phase and composes with all of them. For a user whose complaint is prompt-processing time, the M4→M5 step dominates the whole software list.
>
> The converse also holds. On M4-generation GPUs there is no low-precision datapath at all: Metal 4.1's FP8 matmul is emulated at 0.87–0.94× FP16 throughput (Rigel, 2026), so the "lower the compute roof with FP8/INT8" tier of that page does not apply to M4 Macs.

The same generation step shows up on the [[weight-streaming]] page from the other side: the flash-moe engine runs 3.3× faster on an M5 Max than an M3 Max on identical code.

## Related

- [[mlx]] · [[apple]] · [[quantization-performance]] — the Apple Silicon prefill/decode measurements this extends.
- [[arithmetic-intensity]] · [[hardware-accelerator]] · [[edge-inference]]
- [[methods-to-speed-up-prompt-processing]] · [[sparse-prefill]] · [[prefix-caching]]

## References

- [Exploring LLMs with MLX and the Neural Accelerators in the M5 GPU](https://machinelearning.apple.com/research/exploring-llms-mlx-m5), Apple Machine Learning Research, 2025-11-19.
