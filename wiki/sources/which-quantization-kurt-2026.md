---
title: "Which Quantization Should I Use? A Unified Evaluation of llama.cpp Quantization on Llama-3.1-8B-Instruct"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2601.14277"
year: 2026
authors:
  - Uygar Kurt
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Which Quantization Should I Use? A Unified Evaluation of llama.cpp Quantization

**Kurt** — [arXiv:2601.14277](https://arxiv.org/abs/2601.14277), 2026.

## Summary

A single-model, single-machine sweep of **13 [[gguf|GGUF]] formats** on Llama-3.1-8B-Instruct, reporting file size, perplexity, benchmark accuracy, and `pp512`/`tg128` throughput together. Valuable because it holds everything constant — most quantization comparisons vary model and hardware simultaneously.

Hardware: dual Intel Xeon Platinum 8488C, 96 cores / 192 threads, AVX-512 + BF16 (CPU inference).

## Speed: the two columns tell different stories

| Scheme | pp512 (t/s) | tg128 (t/s) |
|---|---|---|
| F16 | 79.57 | 2.83 |
| Q8_0 | 71.42 | 5.03 |
| Q6_K | 59.81 | 6.33 |
| Q5_1 | **45.98** | 6.33 |
| Q4_K_M | 87.70 | 5.12 |
| Q4_0 | **97.35** | 4.36 |
| Q3_K_S | 57.39 | **9.91** |

**`tg128` is physics**: 2.83 -> 9.91 t/s (**3.5x**), cleanly correlated with file size, exactly as the bandwidth-bound model predicts.

**`pp512` is software**: non-monotonic and uncorrelated with bit width. Q4_0 is fastest (97.35), yet higher-precision Q5_1 manages only 45.98 and Q3_K_S is **28% slower than F16** despite being the smallest. Prefill on CPU is dominated by per-format SIMD kernel maturity — formats with mature AVX-512 GEMM paths win.

> [!warning] Do not read pp512 as a bit-width trend
> These numbers are strong evidence that quantization *does not reliably help prefill*, but they reflect kernel quality rather than roofline physics. See [[quantization-performance]].

## Quality

Perplexity 7.32 (F16) -> 7.33 (Q8_0) -> 7.56 (Q4_K_M) -> 7.96 (Q3_K_M) -> 8.96 (Q3_K_S).
Average benchmark score 69.47 (F16) vs 69.15 (Q4_K_M) vs 65.49 (Q3_K_S).

**The cliff is between 4-bit and 3-bit** — Q4_K_M is within 0.3 points of F16; Q3_K_S loses 4 points.

## Concepts

- [[quantization-performance]] · [[quantization-quality]] · [[gguf]] · [[llama-cpp]]

## References

- [arXiv:2601.14277](https://arxiv.org/abs/2601.14277)
