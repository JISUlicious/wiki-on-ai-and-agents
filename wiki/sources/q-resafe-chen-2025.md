---
title: "Q-resafe: Assessing Safety Risks and Quantization-aware Safety Patching for Quantized Large Language Models"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2506.20251"
venue: ICML 2025
year: 2025
authors:
  - Kejia Chen
tags:
  - 2025
  - paper
status: complete
importance: high
---

# Q-resafe: Safety Risks and Quantization-aware Safety Patching

**Chen et al.** — [arXiv:2506.20251](https://arxiv.org/abs/2506.20251), **ICML 2025**.

## Measured safety loss

Attack Success Rate, full precision -> INT4:

| Setting | Llama-2-7B-Chat | Gemma-7B-Instruct |
|---|---|---|
| Full precision | **0.3** | 9.2 |
| AWQ INT4 | 42.4 | 17.9 |
| AQLM INT4, *benign* calibration | 18.5 | 25.3 |
| AQLM INT4, *harmful* calibration | **77.4** | **55.4** |
| QLoRA INT4, harmful calibration | **85.3** | **68.6** |

## The finding that matters

**The calibration dataset dominates.** The same model at the same bit width spans 0.3 -> 77.4 ASR depending only on what data the quantizer was calibrated with. A quantizer fed harmful calibration data blows alignment wide open.

This makes calibration provenance a **safety property** of any distributed quantized artifact - and it is essentially never documented in community model cards. See [[quantization-safety]].

## Mitigation

Q-resafe applies post-hoc **quantization-aware safety patching** to the already-quantized model:

| | Unpatched | Patched |
|---|---|---|
| Benign calibration | +16.6% / +11.5% over FP baseline | **+1.5% / +0.9%** |
| Harmful calibration | up to +92.3% / +66.7% | **+13.6% / +1.8%** |

## Concepts

- [[quantization-safety]] · [[quantization]] · [[rlhf]]

## References

- [arXiv:2506.20251](https://arxiv.org/abs/2506.20251)

