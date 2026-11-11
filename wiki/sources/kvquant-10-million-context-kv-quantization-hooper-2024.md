---
title: "KVQuant: Towards 10 Million Context Length LLM Inference with KV Cache Quantization"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - kvquant-10-million-context-kv-quantization-hooper-2024.md
tags:
  - 2024
  - paper
status: complete
importance: medium
authors:
  - Coleman Hooper
  - Sehoon Kim
  - Hiva Mohammadzadeh
  - Michael W. Mahoney
  - Yakun Sophia Shao
  - Kurt Keutzer
  - Amir Gholami
introduces:
  - [[kv-cache-quantization]]
year: 2024
venue: NeurIPS 2024
arxiv_id: "2401.18079"
---

# KVQuant: Towards 10 Million Context Length LLM Inference with KV Cache Quantization

[arXiv:2401.18079](https://arxiv.org/abs/2401.18079)

| | |
|---|---|
| **Authors** | Hooper, Kim, Mohammadzadeh, Mahoney, Shao, Keutzer, Gholami |
| **Affiliations** | UC Berkeley, ICSI, LBNL |
| **Venue** | **NeurIPS 2024** |
| **Code** | github.com/SqueezeAILab/KVQuant |

## Summary

KVQuant combines five techniques for [[kv-cache-quantization]] below 4 bits. At 3 bits it loses less than 0.1 perplexity, and it makes **LLaMA-7B usable at 1M context on one A100-80GB and 10M context on 8 GPUs**.

## The five techniques

1. **Per-channel keys** (values stay per-token): **−3.82 PPL** on Wikitext-2 for 3-bit LLaMA-7B. Scales are calibrated **offline**, so there is no grouping and no fp16 residual window, unlike [[kivi-2bit-kv-cache-quantization-liu-2024|KIVI]].
2. **Pre-RoPE key quantization:** RoPE rotates channel pairs and blurs the outlier channels, so keys are quantized *before* RoPE, which is applied on the fly after dequantization (**−0.82 PPL**).
3. **nuqX:** a per-layer, non-uniform datatype found offline by [[k-means]] weighted by Fisher information (**−0.29 PPL** against 3-bit uniform). Dequantization cost is hidden because KV loading is bandwidth-bound.
4. **Per-vector dense-and-sparse:** the top **1%** of outliers are stored in fp16 in sparse format, a further −0.19 PPL, ending "within 0.07 perplexity of the fp16 baseline".
5. **Sink-aware:** the first token is kept in fp16 ([[attention-sink]]); this matters most at 2 bits.

## Results

| Bits | PPL degradation vs fp16 | Memory saving |
|---|---|---|
| 4 | < 0.02 | 3.7× |
| 3 | < 0.1 | 4.8× |
| 2 | < 0.5 | 6.9× |

- **LLaMA-7B perplexity:** fp16 5.68 → 3-bit-1% 5.75 → 2-bit-1% 6.01. Plain int3 gives 10.87 and int2 11,779.
- **Cache size, LLaMA-7B at 128K:** 64 GB in fp16 → 12.0 GB at 3 bits.
- **Long context:** at 10M tokens, nuq2 needs 611.5 GB against 4,882.8 GB in fp16. LLaMA-65B reaches 1M tokens with nuq3 on 8 GPUs.
- **Retrieval:**
  - Passkey (LLaMA-2-7B-32K): about 1.0 at 2.33 bits, against KIVI-2's 0.68–0.76 at 3.05 bits.
  - RULER: 3-bit KVQuant beats KIVI by 14%. Averages are 56.40 fp16 / 53.65 KVQuant-3bit / 39.78 KIVI.
- **Kernels:** about 1.7× faster matvecs (A6000, batch 1).
- **With weight quantization:** combining with SqueezeLLM adds +0.02 PPL (w4) or +0.1 PPL (w3).

## Limitations (Appendix S)

- **Inference only;** training at long context is separate work.
- **Latency measured for generation only,** not prefill.
- **Sparse outlier updates are inefficient,** because old data is copied on each concatenation.
- **Key scales need offline calibration.**

## Related

[[kv-cache-quantization]] · [[kv-cache]] · [[quantization-performance]] · [[attention-sink]] · [[kivi-2bit-kv-cache-quantization-liu-2024]] · [[rope-scaling]]
