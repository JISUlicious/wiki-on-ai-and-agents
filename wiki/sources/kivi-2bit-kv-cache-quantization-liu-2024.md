---
title: "KIVI: A Tuning-Free Asymmetric 2bit Quantization for KV Cache"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - kivi-2bit-kv-cache-quantization-liu-2024.md
tags:
  - 2024
  - paper
status: complete
importance: medium
authors:
  - Zirui Liu
  - Jiayi Yuan
  - Hongye Jin
  - Shaochen (Henry) Zhong
  - Zhaozhuo Xu
  - Vladimir Braverman
  - Beidi Chen
  - Xia Hu
introduces:
  - [[kv-cache-quantization]]
year: 2024
venue: ICML 2024
arxiv_id: "2402.02750"
---

# KIVI: A Tuning-Free Asymmetric 2bit Quantization for KV Cache

[arXiv:2402.02750](https://arxiv.org/abs/2402.02750)

| | |
|---|---|
| **Authors** | Liu, Yuan (equal contribution), Jin, Zhong, Xu, Braverman, Chen, Hu |
| **Affiliations** | Rice, Texas A&M, Stevens, CMU |
| **Venue** | **ICML 2024** |
| **Code** | github.com/jy-yuan/KIVI |

## Summary

A study of what KV-cache values look like, turned into a plug-and-play 2-bit quantizer that needs no tuning ([[kv-cache-quantization]]):
- **Keys have a few fixed outlier channels** ("a few fixed channels whose magnitudes are very large"), so they are quantized **per-channel**.
- **Values have no channel outliers.** Attention output is a sparse weighted mix of value rows (84.3% sparsity), so values are quantized **per-token**, which keeps error confined to each token.

Getting this choice wrong is costly:
- per-token keys give about 5× the attention-score error (47.00 vs 9.60);
- per-channel values give about 15× the output error (49.89 vs 3.55);
- on Llama-2-13B, per-channel values collapse CoQA to about 2.8.

## Mechanism

- **Quantizer:** round-to-nearest asymmetric integer (zero-point = min, scale = (max − min)/(2^B − 1)). GPTQ-style optimization doesn't fit, because the cache is streamed.
- **Streaming layout:**
  - **Keys** are quantized in groups of **G = 32** tokens. A **full-precision residual window** holds up to **R = 128** recent tokens; once full, it is quantized and appended.
  - **Values** use a FIFO queue of length R, quantized per-token as they leave it.
  - The window is "crucial" on hard tasks such as GSM8K.
- **Bits:** 2 (KIVI-2) or 4 (KIVI-4). Prefill passes exact K/V onward and stores only the quantized cache. It combines with weight-only quantization.

## Results

- **Memory and throughput** (Llama-2-7B, one A100-80GB, ShareGPT-like load): **2.6× less peak memory** (weights included), **up to 4× larger batch**, **2.35–3.47× throughput**. The gain is *indirect*: it comes from fitting more sequences ([[quantization-performance]]).
- **Accuracy:** "up to 2% accuracy drop" on GSM8K. Llama-2-7B scores 13.50 → 12.74; Mistral-7B 38.36 → 36.01.
- **LongBench averages:** Llama-2-7B 44.52 → 44.27; Mistral-7B 46.58 → 45.85.
- **Retrieval:** needle-in-a-haystack retrieval holds at 2 bits.

## Limitations

- **Already-compressed caches.** Falcon-7B ([[multi-query-attention|MQA]]) needs 4 bits; 2 bits "may have a large accuracy drop".
- **Throughput headroom.** Fusing quantization with earlier operations is left as future work.
- **Critique from KVQuant.** [[kvquant-10-million-context-kv-quantization-hooper-2024|KVQuant]] argues the fp16 recent window favours the end of the context. On passkey retrieval KIVI-2 scores about 0.68–0.76, against about 1.0 for KVQuant at fewer bits.

## Related

[[kv-cache-quantization]] · [[kv-cache]] · [[quantization-performance]] · [[kvquant-10-million-context-kv-quantization-hooper-2024]] · [[kv-cache-size-vs-active-parameters]]
