---
title: "QuaRot: Outlier-Free 4-Bit Inference in Rotated LLMs"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2404.00456"
venue: NeurIPS 2024
year: 2024
authors:
  - Saleh Ashkboos
  - Amirkeivan Mohtashami
  - Dan Alistarh
  - Torsten Hoefler
  - James Hensman
tags:
  - 2024
  - paper
status: complete
importance: high
---

# QuaRot: Outlier-Free 4-Bit Inference in Rotated LLMs

**Ashkboos, Mohtashami, ..., Alistarh, Hoefler, Hensman** — [arXiv:2404.00456](https://arxiv.org/abs/2404.00456), **NeurIPS 2024**.

## The idea

Instead of special-casing outliers, **change the basis so they cannot exist**.

QuaRot applies **computational-invariance rotations** (Hadamard/orthogonal) to the [[residual-stream|residual stream]], FFN activations, parts of attention, and the [[kv-cache|KV cache]]. A random orthogonal transform spreads a vector's energy evenly across the rotated basis with high probability, so **no channel is special** — while leaving the network's output unchanged. Rotations are folded into adjacent weight matrices where possible; the rest run as cheap online Hadamard transforms.

This is the culmination of the rotation lineage: QuIP -> QuaRot -> SpinQuant.

## Results

**W4A4KV4 — end-to-end 4-bit, all matmuls, with no channels retained in higher precision.** Previously not achievable.

WikiText-2 perplexity: Llama2-7B **6.10**, 13B 5.40, 70B **3.79** — "at most 0.47 perplexity loss and retains 99% of zero-shot performance" for 70B.

Up to **3.33x prefill speedup** (batch 64, seq 2048) and **3.89x decoding memory saving**. Note that prefill speedup is possible here precisely because this is W4A4 — activations are low precision too, so it lowers the compute roof, unlike weight-only schemes ([[quantization-performance]]).

Rotating Llama-2-70B takes 5 minutes on one A100; the GPTQ weight step then takes ~2h. With plain round-to-nearest instead, QuaRot needs **zero calibration data** and is still lossless at 6/8-bit.

> [!note] Evaluation caveat
> QuaRot (and SpinQuant) calibrate on **WikiText-2 train** while also evaluating WikiText-2 perplexity — a same-distribution setup that GPTQ (C4), AWQ/SmoothQuant (Pile) and AQLM/QuIP# (RedPajama) deliberately avoid.

## Concepts

- [[quantization]] — the "rotate them away" family.
- [[llm-int8-dettmers-2022]] — the outlier problem being dissolved.

## References

- [arXiv:2404.00456](https://arxiv.org/abs/2404.00456)
