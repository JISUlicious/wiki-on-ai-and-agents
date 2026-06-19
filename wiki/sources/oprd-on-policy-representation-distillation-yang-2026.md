---
title: "OPRD: On-Policy Representation Distillation"
type: source
created: 2026-06-19
updated: 2026-06-19
sources:
  - oprd-on-policy-representation-distillation-yang-2026.md
arxiv_id: "2606.06021"
authors:
  - Shenzhi Yang
  - et al.
first_author: Shenzhi Yang
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# OPRD: On-Policy Representation Distillation

Surfaced via the alphaXiv weekly digest (2026-06-10).

## Summary

On-policy distillation (OPD) is a cornerstone of LLM post-training, but every existing variant — sampled-token, full-vocabulary, top-k — supervises the student only in the *output space* by matching next-token log-probabilities, treating the teacher as a black-box probability oracle. The authors from [[zhejiang-university]] and [[ant-group]] argue this is doubly limiting: the dominant sampled-token reward is a single-sample Monte-Carlo KL estimate whose gradient variance does not vanish late in training (and top-k removes variance only by introducing truncation bias), and output-space objectives discard the entire stack of intermediate hidden states the teacher actually computed.

OPRD lifts on-policy distillation into the *hidden-state* space: it aligns the student's intermediate representations with the teacher's across selected layers and response positions on the *same* on-policy rollouts, via a layer-averaged, position-masked MSE loss computed before the LM head. This yields dense, deterministic supervision, theoretically eliminates OPD's sampling variance, and exposes per-layer structural information that any output-space objective necessarily throws away — at no extra rollout cost. On three competition-math benchmarks (AIME 2024, AIME 2025, AIMO) OPRD closes the student-teacher gap while output-space OPD baselines plateau several points short, delivering a 2.7-pt accuracy gain, 1.44x faster training, and up to 54% less actor-update peak GPU memory than top-k OPD.

## Key points

- First method to do representation-level knowledge distillation in the *on-policy* regime, complementing (not replacing) output-space OPD — an orthogonal axis of supervision.
- Output-space OPD limits identified: non-vanishing gradient variance (sampled-token) and truncation bias ignoring tail tokens (top-k), plus discarding teacher hidden states (softmax projects through an ill-conditioned W_head).
- Loss: layer-averaged, position-masked MSE between student and teacher hidden states on student-generated trajectories, bypassing the LM head.
- Theory: provably eliminates the sampling variance of the OPD gradient estimator and surfaces per-position, per-layer teacher structure.
- Results: 2.7-pt average accuracy gain over output-space OPD; closes the gap to teacher on AIME24/AIME25/AIMO (Qwen-series students, |V| ~ 150K).
- Efficiency: 1.44x faster training and up to 54% lower actor-update transient memory vs top-k OPD; strictly Pareto-dominant on accuracy, time, and memory.

## Concepts & entities

- [[zhejiang-university]], [[ant-group]] — author affiliations
- Related: [[rethinking-on-policy-distillation-li-2026]] — adjacent work on on-policy distillation
- Related: knowledge distillation, on-policy distillation, RLHF post-training (plain text — no dedicated pages)

## References

- [arXiv:2606.06021](https://arxiv.org/abs/2606.06021)
- Code: github.com/ShenzhiYang2000/OPRD
