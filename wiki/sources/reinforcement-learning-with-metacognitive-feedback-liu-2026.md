---
title: "Reinforcement Learning with Metacognitive Feedback Elicits Faithful Uncertainty Expression in LLMs"
type: source
created: 2026-07-06
updated: 2026-07-06
sources:
  - reinforcement-learning-with-metacognitive-feedback-liu-2026.pdf
arxiv_id: "2606.32032"
authors:
  - Gabrielle Kaili-May Liu
  - Avi Caciularu
  - Gal Yona
  - Idan Szpektor
  - Arman Cohan
first_author: Gabrielle Kaili-May Liu
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Reinforcement Learning with Metacognitive Feedback Elicits Faithful Uncertainty Expression in LLMs

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-28–07-05).

## Summary

LLMs exhibit systemic metacognitive deficiencies: they hallucinate with high confidence, fail to recognize knowledge boundaries, and misrepresent their internal uncertainty. This Yale + Google Research paper posits that models capable of accurately judging their own performance are better positioned to improve it, so it turns **metacognition into the training signal**. It introduces **RLMF (reinforcement learning with metacognitive feedback)**, which refines completion rankings during preference optimization based on the quality of the model's self-judgments of performance, plus **metacognitive data selection** for identifying high-value training examples. The target task is **faithful calibration (FC)** — aligning expressed uncertainty with intrinsic uncertainty, distinct from factual calibration.

## Key points

- **RLMF** rewards the model not only for strong outputs but for accurately judging how well it performed, via a novel **metacognitive advantage scaling** mechanism applied during RL.
- A two-stage decoupled pipeline: first calibrate faithfulness of self-reported numerical confidence scores, then map them to natural, context-adaptable linguistic uncertainty via targeted output editing.
- RLMF **surpasses standard RL by up to 63%** on post-training results while improving models' ability to assess and express their own capability limits.
- Achieves **state-of-the-art faithful calibration** across diverse tasks despite training on a single dataset, while preserving task accuracy and factual calibration (unlike prior methods).
- Human evaluation shows an average **96% win rate** over the strongest baseline on diversity and quality.
- Positions metacognitive performance as an effective RL signal that overcomes limits of prior intrinsic-feedback methods; code released at github.com/yale-nlp/RLMF.

## Concepts & entities

[[reinforcement-learning]] · [[hallucination]] · [[reward-modeling]] · [[google]]

## References

- [arXiv:2606.32032](https://arxiv.org/abs/2606.32032)
