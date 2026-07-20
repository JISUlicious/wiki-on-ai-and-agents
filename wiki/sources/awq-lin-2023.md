---
title: "AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2306.00978"
venue: MLSys 2024 (Best Paper)
year: 2023
authors:
  - Ji Lin
  - Jiaming Tang
  - Haotian Tang
  - Song Han
tags:
  - 2023
  - paper
status: complete
importance: high
---

# AWQ: Activation-aware Weight Quantization

**Lin, Tang, Tang, ..., Han** — [arXiv:2306.00978](https://arxiv.org/abs/2306.00978), **MLSys 2024 (Best Paper)**.

## Mechanism

Not all weights matter equally: protecting roughly **1% of "salient" weight channels** dramatically reduces [[quantization]] error. The key insight is that saliency is identified by **activation magnitude, not weight magnitude** — which channels see large inputs, not which hold large values.

Rather than keeping those channels in higher precision (hardware-hostile mixed precision), AWQ applies a **mathematically equivalent per-input-channel scaling**: scale salient channels up before rounding and fold the inverse into the preceding op. The scale comes from offline activation statistics with a scalar alpha controlling aggressiveness, found by grid search. No backpropagation, no reconstruction.

**Type**: W-only INT3/INT4, group size 128.
**Calibration**: very little. AWQ matches GPTQ quality with **16 sequences vs GPTQ's 192** (10x less).

## Results

- INT4-g128 WikiText-2: LLaMA-7B **5.78**, 13B 5.19, 30B 4.21, 65B 3.62.
- **Calibration robustness** — the practically important result: swapping the calibration distribution (PubMed <-> Enron) costs AWQ **0.5-0.6 PPL** versus GPTQ's **2.3-4.9**. Because AWQ only needs activation *scale* statistics rather than a full second-order fit, it overfits the calibration set far less.
- TinyChat kernels: **2.7-3.9x over HF FP16 on RTX 4090**; 33 tok/s for Llama-2 on Jetson Orin.

> [!note] Calibration size caveat
> The widely-quoted "128 samples of 512 tokens" default comes from the released code, **not the paper** — the paper says only "a small calibration set from the Pile".

## Concepts

- [[quantization]] · [[gptq-frantar-2022]] · [[quantization-quality]]

## References

- [arXiv:2306.00978](https://arxiv.org/abs/2306.00978)
