---
title: "QLoRA: Efficient Finetuning of Quantized LLMs"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2305.14314"
venue: NeurIPS 2023 (Oral)
year: 2023
authors:
  - Tim Dettmers
  - Artidoro Pagnoni
  - Ari Holtzman
  - Luke Zettlemoyer
introduces:
  - [[quantization]]
tags:
  - 2023
  - paper
status: complete
importance: high
---

# QLoRA: Efficient Finetuning of Quantized LLMs

**Dettmers, Pagnoni, Holtzman, Zettlemoyer** — [arXiv:2305.14314](https://arxiv.org/abs/2305.14314), **NeurIPS 2023 (Oral)**.

## NF4 (NormalFloat4)

A **quantile quantization** data type built from the quantiles of a standard normal, motivated by the observation that pretrained weights are approximately zero-centered normal — so an information-theoretically optimal grid for `N(0,1)` beats a uniform grid.

The subtle part: symmetric k-bit quantile quantization has **no exact representation of zero**, which is bad for padding and zero-valued elements. So they build an **asymmetric** grid — estimate `2^(k-1)` quantiles for the negative side and `2^(k-1)+1` for the positive, union them and drop the duplicate zero — guaranteeing an exact zero while using all `2^k` codes.

## Double Quantization

Scale metadata is a real cost: FP32 scales at blocksize 64 add `32/64 = 0.5 bits/param`. DQ quantizes those constants with 8-bit floats at blocksize 256, cutting overhead to `8/64 + 32/(64*256) = 0.127 bits/param` — a **0.373 bit/param saving**.

This is the canonical illustration of the granularity/metadata tradeoff in [[quantization]], and the same two-level idea appears in llama.cpp's [[gguf|k-quant super-blocks]].

**Type**: W-only 4-bit storage, dequantized to BF16 for the matmul — a *memory* technique, not a compute speedup.
**Calibration**: none; data-free blockwise absmax.

## Results

Finetunes a **65B model on a single 48GB GPU** while preserving full 16-bit finetuning task performance. Guanaco-65B reaches **99.3% of ChatGPT** on the Vicuna benchmark after 24h on one GPU. NF4 + DQ fully recovers 16-bit performance; NF4 > FP4 at equal bits.

## Concepts

- [[quantization]] · [[lora]] · [[fine-tuning]]

## References

- [arXiv:2305.14314](https://arxiv.org/abs/2305.14314)
