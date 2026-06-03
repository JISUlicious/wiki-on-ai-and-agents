---
title: "Self-Distilled RLVR"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - self-distilled-rlvr-yang-2026.md
arxiv_id: "2604.03128"
authors:
  - Chenxu Yang et al.
first_author: Chenxu Yang
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Self-Distilled RLVR

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

This paper (CAS Institute of Information Engineering + JD.COM) studies the interaction between **on-policy self-distillation (OPSD)** and **RLVR** (RL with verifiable rewards). In OPSD the same model is both teacher and student, with the teacher given privileged information (e.g. reference answers) for self-evolution. The authors show, with controlled experiments and a formal mutual-information analysis, that learning *solely* from the privileged teacher causes severe information leakage: the distribution-matching objective is structurally ill-posed under teacher/student asymmetry, producing early gains followed by systematic collapse and unstable long-term training.

Their fix, **RLSD (RLVR with Self-Distillation)**, decouples the two signals: the verifiable environment reward governs the *direction* of updates, while the teacher/student discrepancy is repurposed to modulate the *per-token magnitude* of updates. It acts as a drop-in replacement for GRPO's uniform advantage, with no auxiliary loss or extra model. RLSD achieves a higher convergence ceiling and better stability, posting the best average accuracy across five multimodal reasoning benchmarks (+4.69% over the base LLM on Qwen3-VL-8B-Instruct) and reaching at 200 steps what GRPO needs 400 steps for.

## Key points

- Diagnoses OPSD failure: information asymmetry → irreducible mutual-information gap → privileged-information leakage through gradient structure → collapse after early gains.
- RLSD splits the signal: RLVR sets *update direction* from correctness; self-distillation sets *update magnitude* per token.
- Drop-in replacement for GRPO's uniform advantage; no auxiliary loss, no separate teacher model.
- Higher convergence ceiling and stability than both OPSD and plain GRPO.
- +4.69% over base on five multimodal reasoning benchmarks; matches 2x-step GRPO at half the steps.
- Preliminary gains also reported on text reasoning, video understanding, and non-Qwen models (deferred to a future version).

## Concepts & entities

[[reinforcement-learning]] · [[grpo]] · [[ppo]] · [[chain-of-thought]] · [[fine-tuning]] · [[reward-modeling]] · [[multimodal-llm]] · [[deepseek-r1]]

## References

- arXiv: [arXiv:2604.03128](https://arxiv.org/abs/2604.03128)
