---
title: "Beyond Scalar Rewards by Internalizing Reasoning into Score Distributions"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - beyond-scalar-rewards-z-reward-jin-2026.pdf
arxiv_id: "2606.09076"
authors:
  - Xin Jin
  - Huanqia Cai
  - Zhen Li
  - Zechao Zhan
  - Chunle Guo
  - Peng Gao
  - Steven C.H. Hoi
first_author: Xin Jin
year: 2026
venue: arXiv preprint (Alibaba Z-Image Team / Nankai University)
tags: [2026]
status: complete
importance: medium
---

# Beyond Scalar Rewards by Internalizing Reasoning into Score Distributions

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-07–06-14).

## Summary

Reward models compress human judgment into a single scalar, but the paper argues visual/aesthetic preference is subjective and better captured as a **distribution over rubric scores** than as a deterministic number. **Z-Reward** is a teacher-student reward-modeling framework (in the text-to-image post-training setting) that decouples reasoning-heavy judgment from efficient deployment. A large VLM **teacher** reasons to infer rubric-aligned score *distributions*, trained with Group-wise Direct Score Optimization (GDSO). A compact VLM **student** then internalizes the teacher's reasoning-conditioned distribution via Reasoning-Internalized Score Distillation (RISD), so it predicts the distribution without producing explicit reasoning chains at inference time.

## Key points

- **Thesis:** scalar (and score-token / pairwise) reward models over-compress uncertainty and fine-grained score gaps; preferences are better modeled as score distributions.
- **Teacher (GDSO):** a large VLM reasons to rubric-aligned score distributions; GDSO mixes policy-gradient rewards from distribution expectations with pointwise/pairwise supervision on distributions and score gaps.
- **Student (RISD):** distills the teacher's reasoning-conditioned distribution into a compact VLM, dropping explicit reasoning at inference.
- **Results:** 27B GDSO teacher reaches **89.6%** human-preference accuracy (beating SFT, RewardDance, GRPO); 9B RISD student reaches **88.6%**, beating an OPD baseline and nearly matching the teacher.
- As a differentiable reward for text-to-image optimization, Z-Reward yields a **+41.3%** net human-preference improvement over the SFT baseline.

## Concepts & entities

- [[reinforcement-learning]] — Z-Reward serves as a (differentiable) reward signal for post-training optimization.
- [[knowledge-distillation]] — RISD distills the teacher's score distribution into a compact student.
- [[reward-modeling]] — the paper rethinks reward models as distribution predictors rather than scalar scorers.

## References

- [arXiv:2606.09076](https://arxiv.org/abs/2606.09076)
- Source PDF: `beyond-scalar-rewards-z-reward-jin-2026.pdf`
