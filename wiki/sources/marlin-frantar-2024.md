---
title: "MARLIN: Mixed-Precision Auto-Regressive Parallel Inference on Large Language Models"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2408.11743"
venue: PPoPP 2025
year: 2024
authors:
  - Elias Frantar
  - Roberto L. Castro
  - Jiale Chen
  - Torsten Hoefler
  - Dan Alistarh
tags:
  - 2024
  - paper
status: complete
importance: high
---

# MARLIN: Mixed-Precision Auto-Regressive Parallel Inference

**Frantar, Castro, Chen, Hoefler, Alistarh** — [arXiv:2408.11743](https://arxiv.org/abs/2408.11743), **PPoPP 2025**.

## Summary

A W4A16 GEMM kernel that holds near-ideal speedup up to **batch 32**, where prior 4-bit kernels degrade almost immediately. Its value for this wiki is less the engineering than the **precise quantification of when weight-only [[quantization]] stops paying off**.

## The crossover

> "For 4bit-weight inference, MARLIN obtains speedups of approximately **3.9x** relative to FP16 ... for batch sizes of up to **16-32**. Speedups gradually reduce, towards **1.5x at batch size 128**, as the problem becomes compute-bound."

The design principle behind it is the single best statement of the tradeoff:

> "Modern GPUs ... typically have a FLOP-to-byte ratio in the range of **100 to 200** for FP16 operations. Thus, if one would be able to reduce weight precision to 4 bits while maintaining a proportional number of multiply-accumulate operations per quantized weight (in this case, in the range of **25-50**), one could theoretically still obtain close to the optimal 4x speedup."

So **W4A16 wins exactly while you perform fewer than ~25-50 MACs per 4-bit weight**. Above that, you are on the compute roof and 4-bit weights buy nothing.

The "ideal 3.87x" figure is not arbitrary: it is `16 / 4.125`, the compression of a symmetric group-128 4-bit format including its FP16 scale.

## Techniques

Asynchronous global weight loads, a circular shared-memory queue, striped partitioning, and careful interleaving of dequantization with tensor-core instructions. Up to **2.8x end-to-end** integrated into [[vllm|vLLM]]; Sparse-MARLIN (2:4 sparsity) adds up to **+65%**.

A useful methodological note: at *locked base GPU clocks* — the realistic production setting — competing kernels' relative speedups degrade significantly while Marlin's do not. Most published kernel benchmarks run at boost clocks.

## Concepts

- [[quantization-performance]] — batch-size regimes.
- [[quantization]] · [[vllm]]

## References

- [arXiv:2408.11743](https://arxiv.org/abs/2408.11743) — MARLIN (PPoPP 2025)
