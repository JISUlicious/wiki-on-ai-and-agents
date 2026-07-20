---
title: "SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2211.10438"
venue: ICML 2023
year: 2022
authors:
  - Guangxuan Xiao
  - Ji Lin
  - Mickael Seznec
  - Hao Wu
  - Julien Demouth
  - Song Han
tags:
  - 2022
  - paper
status: complete
importance: high
---

# SmoothQuant

**Xiao, Lin, Seznec, Wu, Demouth, Han** — [arXiv:2211.10438](https://arxiv.org/abs/2211.10438), **ICML 2023**.

## The problem it solves

Weights are easy to quantize; activations are not, because of persistent **outlier channels** ([[llm-int8-dettmers-2022]]). And the natural fix is not available: for activations the outliers live along the *channel* axis, which **is the GEMM's reduction axis**, so a per-channel activation scale cannot factor out of the inner product. The paper states this directly — per-channel activation quantization would fix accuracy but is incompatible with INT8 GEMM kernels, while per-token quantization is hardware-legal but "helps little".

## Mechanism

Apply a **mathematically equivalent per-channel transformation**: divide activations by `s_j`, multiply the corresponding weight column by `s_j`. This **migrates quantization difficulty from activations into weights, offline**, so both become quantizable.

`s_j = max(|X_j|)^alpha / max(|W_j|)^(1-alpha)`, where **migration strength alpha = 0.5** is the sweet spot for all OPT and BLOOM models (0.75 for the harder GLM-130B).

**Type**: **W8A8** — INT8 weights *and* activations. This is the point: weight-only quantization saves memory but not compute; W8A8 unlocks integer GEMM kernels.
**Calibration**: 512 random sentences from the Pile, used once.

## Results

Average accuracy (SmoothQuant-O3 vs FP16): OPT-175B **74.6% vs 71.6%**, BLOOM-176B 67.4% vs 68.2%, GLM-130B 72.8% vs 73.8%. Up to **1.51x speedup / 1.96x memory saving**; MT-NLG 530B served on a single 8-GPU node (527GB vs 1040GB).

## Concepts

- [[quantization]] — the "move the outliers" family.
- [[quantization-performance]] — why W8A8 differs from weight-only.

## References

- [arXiv:2211.10438](https://arxiv.org/abs/2211.10438)
