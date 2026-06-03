---
title: "Vector Policy Optimization: Training for Diversity Improves Test-Time Search"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - vector-policy-optimization-bahlous-boldi-2026.pdf
arxiv_id: "2605.22817"
authors:
  - Ryan Bahlous-Boldi
  - Idan Shenfeld
  - Omar Khattab
  - Akarsh Kumar
  - Zhang-Wei Hong
  - Mehul Damani
  - Pulkit Agrawal
first_author: Ryan Bahlous-Boldi
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Vector Policy Optimization: Training for Diversity Improves Test-Time Search

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

Vector Policy Optimization (VPO), from MIT and the Improbable AI Lab, is an RL post-training algorithm that explicitly trains policies to produce *diverse* solutions so they perform better inside inference-time search procedures (e.g., AlphaEvolve, best-of-n, evolutionary search). The standard paradigm optimizes a single pre-specified scalar reward, which collapses LLMs toward low-entropy response distributions and undermines the diversity that test-time search needs. VPO exploits the fact that rewards are often *vector-valued* in practice — per-test-case correctness in code generation, multiple user personas, or multiple reward models — and is essentially a drop-in replacement for the GRPO advantage estimator that trains the policy to anticipate diverse downstream reward functions rather than commit to one mode.

## Key points

- Diagnoses scalar-reward RL as a cause of low-entropy, mode-collapsed policies that hurt test-time search.
- Treats rewards as vector-valued (per-test-case, per-persona, or per-reward-model) instead of a single scalar.
- Drop-in replacement for the GRPO advantage estimator; minimal changes to existing RL pipelines.
- Matches or beats strongest scalar RL baselines on pass@k and best@k across four tasks, with the gap widening as search budget grows.
- For evolutionary search, VPO-trained models unlock problems that GRPO models cannot solve.
- Frames training-time diversity as a first-class objective for inference-scaling agents.

## Concepts & entities

- [[reinforcement-learning]]
- [[grpo]]
- [[llm-agent]]

## References

- [arXiv:2605.22817](https://arxiv.org/abs/2605.22817)
