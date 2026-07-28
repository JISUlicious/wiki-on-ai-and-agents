---
title: "8-bit Optimizers via Block-wise Quantization"
type: source
created: 2026-07-29
updated: 2026-07-29
sources:
  - 8-bit-optimizers-dettmers-2021.md
arxiv_id: "2110.02861"
year: 2021
venue: ICLR 2022 (spotlight)
authors:
  - Tim Dettmers
  - Mike Lewis
  - Sam Shleifer
  - Luke Zettlemoyer
tags:
  - 2021
  - paper
status: complete
importance: high
---

# 8-bit Optimizers via Block-wise Quantization

**Dettmers, Lewis, Shleifer, Zettlemoyer** (Facebook AI Research; University of Washington) — [arXiv:2110.02861](https://arxiv.org/abs/2110.02861), **ICLR 2022 spotlight**.

## Summary

This paper attacks the same memory line item as [[zero-redundancy-optimizer]] — the optimizer states — but by compressing them rather than distributing them, and therefore works on a *single* device. Its framing claim is the one most worth citing: **optimizer states use 33-75% of the total memory footprint during training**, and reducing that footprint was, at the time, "much less studied" than reducing parameter memory. Concretely, 32-bit Momentum costs **4 bytes per parameter** and 32-bit Adam **8 bytes per parameter** — 4 GB and 8 GB for a 1B-parameter model. The paper reduces these to **1 GB and 2 GB**, a 4x cut in optimizer state.

Getting to 8 bits means going from 65,536 representable values (fp16) to 256, which the paper solves with three components. **Block-wise dynamic quantization** chunks the state tensor into blocks of **B = 2048** elements and computes a separate normalization constant Nb = max(|Tb|) per block, so a single outlier corrupts only its own block instead of the whole tensor — and because each block normalizes independently, no cross-core synchronization is needed, which is what makes 8-bit optimizers *faster* than 32-bit ones rather than slower. **Dynamic quantization** is a non-linear codebook precise for both large and small magnitudes. A **stable embedding layer** reduces the gradient variance caused by the highly non-uniform token distribution in language models. Dequantize→update→requantize happens element-by-element in registers, so no extra GPU memory traffic or temp buffers.

Quality is preserved, and the ablations show all three components are load-bearing. Across GLUE, ImageNet classification, WMT'14 machine translation, MoCo v2 contrastive pretraining, RoBERTa pretraining, and 1.5B-parameter language modeling, 8-bit optimizers **match or slightly exceed** replicated 32-bit results with **no hyperparameter changes** — while training slightly *faster*. It is a two-line code change (this work is the origin of `bitsandbytes`, later reused for [[qlora-dettmers-2023]]).

## Key points

### What the optimizer states actually cost

| Optimizer | 32-bit state | 8-bit state | 1B-param model |
|---|---|---|---|
| Momentum (1 state) | 4 bytes/param | 1 byte/param | 4 GB → **1 GB** |
| Adam (2 states) | 8 bytes/param | 2 bytes/param | 8 GB → **2 GB** |

Paper's headline framing: *"optimizer states use 33-75% of the total memory footprint during training."* Its examples: Adam states for the largest GPT-2 and T5 models are **11 GB and 41 GB**.

> [!note] Different accounting from ZeRO
> Dettmers et al. count only Adam's two moments (8 bytes/param) as optimizer state, on top of a 16-bit model/gradient representation. [[zero-redundancy-optimizer]] counts 12 bytes/param because it also includes the fp32 master parameter copy. Both are correct within their own scheme — worth reconciling explicitly when comparing.

### Method

1. **Block-wise quantization** — blocks of **B = 2048**; per-block normalization constant Nb = max(|Tb|); n/B blocks per tensor; no cross-core reduction, isolates outliers.
2. **Dynamic quantization** — non-linear codebook, precise across magnitudes.
3. **Stable embedding layer** — cuts gradient variance from non-uniform token frequency (used for all NLP tasks except GLUE finetuning).

### Quality is preserved (Table 1, medians)

| Task / model | 32-bit | 8-bit | Time (32-bit → 8-bit) | Memory saved |
|---|---|---|---|---|
| GLUE, RoBERTa-Large (AdamW) | 88.6 | **88.7** | 17h → 15h | 2.0 GB |
| ImageNet-1k CLS, ResNet-50 (Momentum) | 77.1 | **77.2** | 118h → 116h | 0.1 GB |
| WMT'14+16 MT, Transformer (BLEU) | 29.0 | **29.1** | 126h → 115h | 1.1 GB |
| MoCo v2, ResNet-50 | 67.3 | **67.4** | 30d → 28d | 0.1 GB |
| LM, Transformer-1.5B (ppl) | 9.0 | **9.0** | 308d → 297d | **8.5 GB** |
| LM, GPT3-Medium (ppl) | 10.62 | **10.62** | 795d → 761d | 1.7 GB |
| Masked-LM, RoBERTa-Base | 3.49 | **3.48** | 101d → 94d | 1.1 GB |

32-bit Adafactor is competitive on quality but uses "almost twice as much memory" as 8-bit Adam and trains slower (e.g. 5.6 GB vs 8.5 GB saved on Transformer-1.5B; 316d vs 297d).

### Ablations (Table 3) — every component matters

| Params | Config | Unstable runs | Perplexity |
|---|---|---|---|
| 209M | 32-bit Adam | 0% | 16.7 |
| 209M | 8-bit Adam, linear quant | **90%** | 253.0 |
| 209M | + dynamic quant | 10% | 18.6 |
| 209M | + dynamic + block-wise | **0%** | 16.8 |
| 209M | + dynamic + block-wise + stable emb | **0%** | **16.4** |
| 1.3B | 32-bit Adam | 0% | 10.4 |
| 1.3B | 8-bit Adam, dynamic only | **100%** | N/A |
| 1.3B | 8-bit Adam, dynamic + stable emb | 80% | 10.9 |
| 1.5B | 32-bit Adam | 0% | **9.0** |
| 1.5B | 8-bit Adam, all three | 0% | **9.0** |

Reading: dynamic quantization is what makes 8-bit viable at all; **block-wise quantization is what makes it stable at scale** (1.3B diverges 80-100% of the time without it).

### Accessibility (Table 2) — largest finetunable model at batch size 1

| GPU memory | 32-bit Adam | 8-bit Adam |
|---|---|---|
| 6 GB | RoBERTa-base (110M) | RoBERTa-large (355M) |
| 11 GB | MT5-small (300M) | MT5-base (580M) |
| 24 GB | MT5-base (580M) | MT5-large (1.2B) |
| 24 GB | GPT-2-medium (762M) | GPT-2-large (1.5B) |

## Concepts & entities

- [[training-memory-anatomy]] — supplies the "optimizer states are 33-75% of training memory" figure and the 4 / 8 bytes-per-parameter costs for Momentum / Adam.
- [[quantization]] — block-wise dynamic quantization applied to optimizer state rather than weights or activations; block size 2048.
- [[quantization-performance]] — a rare case where quantization is both lossless in quality and slightly *faster* than the full-precision baseline.
- [[mixed-precision-training]] — assumes the standard 16-bit model/gradient representation and compresses the remaining states on top of it.
- [[llm-int8-dettmers-2022]] — same first author; the outlier problem handled here by block-wise normalization reappears there as emergent outlier features in activations.
- [[qlora-dettmers-2023]] — builds directly on this line of work (and its `bitsandbytes` implementation), pairing 8-bit optimizer state with 4-bit weights.
- [[zero-redundancy-optimizer]] — the complementary approach: distribute optimizer state instead of compressing it. The paper notes 8-bit optimizers are "one of the few options that can reduce the optimizer memory footprint significantly for single devices."
- [[lora]] — the parameter-efficient alternative that shrinks optimizer state by shrinking the number of trainable parameters instead.

## References

- [arXiv:2110.02861](https://arxiv.org/abs/2110.02861)
