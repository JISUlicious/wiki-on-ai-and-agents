---
title: "Scaling Laws for Precision"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2411.04330"
venue: ICLR 2025
year: 2024
authors:
  - Tanishq Kumar
tags:
  - 2024
  - paper
status: complete
importance: high
---

# Scaling Laws for Precision

**Kumar et al.** — [arXiv:2411.04330](https://arxiv.org/abs/2411.04330), **ICLR 2025**.

## The post-training-quantization law

$$\delta_{PTQ}(N, D, P_{post}) = C_T \left(\frac{D^{\gamma_D}}{N^{\gamma_N}}\right) e^{-P_{post}/\gamma_{post}}$$

All coefficients fitted **positive** (R^2 = 0.97). Reading it:

- degradation **grows with training tokens D**
- degradation **shrinks with parameters N**
- degradation **grows exponentially as precision falls**

Since fitted gamma_D ~= gamma_N, it is approximately a power law in the **token/parameter ratio D/N**.

## The counterintuitive finding

*"Overtrained language models are more sensitive to post-training quantization... when D/N is sufficiently large, or P_post sufficiently small, loss after quantization can increase as models are pretrained for longer."*

Their intuition: as models train on more data they **compress more information into their weights**, so perturbing those weights is more damaging.

Taken to its conclusion: *"there exists an amount of pretraining data beyond which additional data is actively harmful to performance at inference-time."* This is a genuinely uncomfortable result for an industry trend of ever-longer training runs — and it is independently corroborated by [[low-bit-favors-undertrained-ouyang-2024]].

## Other findings

- **Compute-optimal pretraining precision is ~7-8 bits**, and (when N, D, P are jointly optimized) independent of compute budget. If N is fixed in advance and the model is overtrained, *higher* precision during training may be compute-optimal.
- **Low-precision training robustifies**: it forces weights that tolerate quantization noise, so low-precision-trained models degrade *less* under subsequent PTQ.
- The law itself implies **larger models are more robust** to weight quantization at fixed data.

> [!warning] Scope
> Fitted on models up to **1.7B parameters / 26B tokens** (465 pretraining runs, GPTQ), then extrapolated aggressively. The authors caution readers *"not to take too seriously the numerical values we fit, but the trends and functional forms we identify"*, citing the failed Chinchilla replication as precedent.

## Concepts

- [[quantization-quality]] · [[scaling-laws]] · [[quantization]]

## References

- [arXiv:2411.04330](https://arxiv.org/abs/2411.04330)

