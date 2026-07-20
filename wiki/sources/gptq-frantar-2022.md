---
title: "GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2210.17323"
venue: ICLR 2023
year: 2022
authors:
  - Elias Frantar
  - Saleh Ashkboos
  - Torsten Hoefler
  - Dan Alistarh
tags:
  - 2022
  - paper
status: complete
importance: high
---

# GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers

**Frantar, Ashkboos, Hoefler, Alistarh** — [arXiv:2210.17323](https://arxiv.org/abs/2210.17323), **ICLR 2023**. (Sometimes cited as **OPTQ**, the ICLR title.)

## Mechanism

Layer-wise weight-only [[quantization]]. For each linear layer it solves `argmin ||WX - WqX||^2` using approximate **second-order information** (the inverse Hessian, where `H = 2XX^T`): quantize one column, then **update all remaining unquantized columns to compensate for the error just introduced**. Descends from Optimal Brain Quantization.

Three engineering changes make it scale to 175B:

1. **Arbitrary/fixed column order** instead of OBQ's greedy per-row ordering
2. **Lazy batch updates** in blocks of 128 columns
3. **Cholesky reformulation** of the inverse Hessian with 1% dampening for numerical stability

**Type**: W-only, 4/3/2-bit; activations stay FP16.
**Calibration**: 128 random 2048-token segments from C4.

## Results

- OPT-175B 4-bit WikiText2 **PPL 8.37 vs 8.34 FP16** — a 0.03 loss.
- Quantizes 175B in **~4 GPU-hours** on a single A100-80GB (OPT-175B 4.2h, BLOOM-176B 3.8h).
- End-to-end **~3.25x vs FP16 on A100**, 4.5x on A6000.

Along with [[awq-lin-2023|AWQ]], GPTQ is one of the two default weight-only PTQ methods in production use, and the basis of the EXL2 format.

## Concepts

- [[quantization]] · [[quantization-quality]] · [[marlin-frantar-2024]] (W4A16 kernels for GPTQ-style weights)

## References

- [arXiv:2210.17323](https://arxiv.org/abs/2210.17323)
