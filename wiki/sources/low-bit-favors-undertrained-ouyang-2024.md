---
title: "Low-Bit Quantization Favors Undertrained LLMs: Scaling Laws for Quantized LLMs with 100T Training Tokens"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2411.17691"
year: 2024
authors:
  - Xu Ouyang
tags:
  - 2024
  - paper
status: complete
importance: high
---

# Low-Bit Quantization Favors Undertrained LLMs

**Ouyang et al.** — [arXiv:2411.17691](https://arxiv.org/abs/2411.17691), 2024.

## Claim

*"Models with larger sizes or fewer training tokens experience less quantization-induced degradation (QiD) when applying low-bit quantization, whereas smaller models with extensive training tokens suffer significant QiD."*

Fitted scaling law: **DeltaqLoss(D) ~= b * D^0.5316**.

Scope: Pythia 160M-12B, 1B-206B tokens, **1,500+ checkpoints**, 2/3/4-bit via GPTQ, AWQ and bitsandbytes. This checkpoint-sweep design is what makes it a genuinely independent test of [[scaling-laws-for-precision-kumar-2024|Kumar et al.]]'s D-dependence rather than a re-derivation.

Extrapolated: QiD is acceptable for models trained on <=10^12 tokens but *"predicted to become undesirable when applied to fully trained LLMs (e.g., trained with 100 trillion tokens)"*, with 2- and 3-bit falling below random-guess level.

## Direct corroboration across model generations

The prediction is visible in released models. Same quantization, near-identical parameter counts, different token budgets:

| Model | Training tokens | q2_K perplexity cost |
|---|---|---|
| Llama-2-7B | 2T | **+10.8%** |
| Llama-3-8B | 15T | **+56.5%** |

Token agreement ("same top-p") falls from 85.6% to **71.1%**. At q4_K_M the gap is small (+1.4% vs +2.8%) - the divergence is a low-bit phenomenon.

Qwen3 shows the same pattern: Qwen3-8B-Base under 3-bit AWQ goes **10.4 -> 23.8** perplexity where LLaMA3-8B goes 9.2 -> 11.6, which the Qwen3 study attributes to *"fewer redundant representations"* from more advanced pretraining.

**Synthesis**: three independent lines - a fitted law, a checkpoint sweep, and cross-generation measurement - agree that **tokens-per-parameter predicts quantization fragility better than parameter count**, and that modern overtrained models are getting harder to quantize.

## Concepts

- [[quantization-quality]] · [[scaling-laws]] · [[quantization]]

## References

- [arXiv:2411.17691](https://arxiv.org/abs/2411.17691)

