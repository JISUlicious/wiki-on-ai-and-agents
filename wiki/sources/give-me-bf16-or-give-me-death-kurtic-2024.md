---
title: "\"Give Me BF16 or Give Me Death\"? Accuracy-Performance Trade-Offs in LLM Quantization"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2411.02355"
year: 2024
authors:
  - Eldar Kurtic
  - Alexandre Marques
  - Shubhra Pandit
  - Mark Kurtz
  - Dan Alistarh
tags:
  - 2024
  - paper
status: complete
importance: high
---

# "Give Me BF16 or Give Me Death"? Accuracy-Performance Trade-Offs in LLM Quantization

**Kurtic, Marques, Pandit, Kurtz, Alistarh** — [arXiv:2411.02355](https://arxiv.org/abs/2411.02355), 2024.

## Summary

The largest systematic quantization study available: **~500,000 evaluations**, vLLM 0.6.1, across A6000 / A100 / H100 and Llama-3.1 8B / 70B / 405B, over six latency-sensitive and two throughput scenarios. It is the best single source for *which format to deploy*, because it resolves the answer by **deployment scenario** rather than reporting one number.

## Accuracy findings

- **W8A8-FP8 is lossless at all model scales.**
- **W8A8-INT**, properly tuned, costs **1-3%**.
- **W4A16-INT** is competitive with 8-bit integer W8A8.

## Performance findings

| Scheme | Compression | Speedup | Wins for |
|---|---|---|---|
| **W4A16-INT** | ~3.5x | **2.4x avg** | single-stream / latency |
| **W8A8-INT** | ~2x | **1.8x avg** | server / throughput (A100) |
| **W8A8-FP8** | ~2x | **1.8x avg** | server / throughput (H100) |

The split is the prefill/decode asymmetry expressed as a deployment question ([[quantization-performance]]):

- **Synchronous / single-stream** is pure batch-1 decode -> bandwidth-bound -> weight bytes are everything. W4A16 wins at every size: 1.5-2x latency improvement for 8B, and **6-7x cost reduction for 405B, letting it run on 4 GPUs instead of 16**.
- **Asynchronous / server** batching crosses the roofline ridge, so only formats that lower the *compute* roof help. W8A8-FP8 best on H100, W8A8-INT best on A100; W4A16 still best on A6000 and for smaller models.

Their own summary: *"the most cost-effective quantization scheme depends on model size, hardware, use case, and deployment scenario."*

## Concepts

- [[quantization-performance]] · [[quantization]] · [[quantization-quality]] · [[vllm]]

## References

- [arXiv:2411.02355](https://arxiv.org/abs/2411.02355)
