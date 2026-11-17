---
title: "When AI Benchmarks Plateau: A Systematic Study of Benchmark Saturation"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - when-ai-benchmarks-plateau-akhtar-2026.md
arxiv_id: "2602.16763"
venue: ICML 2026 (PMLR 306)
authors:
  - Mubashara Akhtar
  - Anka Reuel
  - Prajna Soni
  - Leshem Choshen
  - Mykel J. Kochenderfer
  - Sanmi Koyejo
  - Mrinmaya Sachan
  - Stella Biderman
  - Zeerak Talat
  - Avijit Ghosh
  - Irene Solaiman
  - et al. (37 authors)
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# When AI Benchmarks Plateau: A Systematic Study of Benchmark Saturation

**Source:** Mubashara Akhtar, Anka Reuel (lead authors), … Stella Biderman, Zeerak Talat, Avijit Ghosh, Irene Solaiman (ETH Zurich; [[stanford-university|Stanford]]; [[eleutherai|EleutherAI]]; [[huggingface|Hugging Face]]; [[cohere|Cohere]]; and others) — [arXiv:2602.16763](https://arxiv.org/abs/2602.16763), cs.AI, v1 18 Feb 2026 (v4 6 Aug 2026). Published at ICML 2026. Produced by the EvalEval Coalition.

## Summary

Benchmarks lose value once the top models can no longer be told apart on them. Earlier work describes this "saturation" informally, or ties it to reaching human-level scores. This paper defines it as **the loss of reliable discriminative power among top models** and turns that into an uncertainty-aware **saturation index**.

The index takes the top k = 5 leaderboard scores and divides their spread (s₁ − s_k) by the standard error of that difference. To keep huge test sets from dominating, it uses an effective test-set size n_eff = n^0.5. The result is R_norm, and S_index = exp(−R_norm²) lies in [0, 1]. The paper separates *stagnation* (top models are statistically alike) from *saturation* (alike **and** near the empirical ceiling).

The authors annotate 60 text-only LLM benchmarks on 14 properties. The benchmarks were drawn from 61 model-developer reports (Jan 2022 – Nov 2025) and from highly cited papers. They then test six hypotheses about what drives saturation.

## Key points

- **Saturation is common.** 29 of 60 benchmarks have high or very high saturation (S_index ≥ 0.7), and 14 of those are ≥ 0.9.
- **Age is the main driver.** The saturated share rises from 42.9% for benchmarks under 24 months old to 54.5% for those over 60 months. Mean S_index by age bin is 0.51, 0.52 and 0.60. In a Bayesian regression, age and test-set size have the most consistent effects.
- **Private test sets don't help.** Public (N = 56) and private (N = 4) benchmarks show no meaningful difference, so the authors reject H1. Hiding test data does not prevent saturation once a benchmark is widely adopted.
- **Format and templating don't matter much.** Closed- vs open-ended format (p = 0.40) and templated vs non-templated (p = 0.10) show no reliable effect.
- **Expert curation helps.** Expert-curated benchmarks saturate less than others of the same age, and ARC-AGI and BIG-Bench Hard remain unsaturated. The apparent multilingual advantage is mostly an age effect: multilingual benchmarks average 32.9 months old, against 48.9 for English-only.
- **Adoption is not the cause.** Once age is controlled for, citation count (ρ = 0.22, p = 0.12) and appearances in technical reports (ρ = 0.05, p = 0.73) are not significant.
- **Robust to settings.** The index ranking is stable across k ∈ {3, 5} and α ∈ {0, 0.5, 1}, with Spearman ρ = 0.88–0.92.
- Recommendations: monitor saturation, report uncertainty, and set explicit criteria for retiring or revising a benchmark.

## Concepts & entities

- [[benchmark]] · [[benchmark-contamination]] — saturation as a different failure from contamination. Keeping the test set private does not prevent it.
- [[humaneval]] · [[mmlu]] · [[gsm8k]] · [[swe-bench]] · [[terminal-bench]] — among the 60 benchmarks studied. The paper cites HumanEval as a canonical case of rapid saturation.
- [[agent-evaluation]] · [[stanford-university]] · [[eleutherai]] · [[huggingface]] · [[cohere]].

## References

- [arXiv:2602.16763](https://arxiv.org/abs/2602.16763)
