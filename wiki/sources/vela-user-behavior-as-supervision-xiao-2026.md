---
title: "Vela: User Behavior as Supervision Sparks LLM Intelligence"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - vela-user-behavior-as-supervision-xiao-2026.md
arxiv_id: "2610.14709"
authors:
  - Yang Xiao
  - Yusong Sun
  - Haoyi Wu
  - Chengyue Jiang
  - Wenjie Li
  - Chuan Mu
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Vela: User Behavior as Supervision Sparks LLM Intelligence

**Yang Xiao, Yusong Sun** (equal contribution) *et al.*, 20 authors under "AllSpark Team". Chengyue Jiang is project lead; the contact is a PolyU address. [arXiv:2610.14709](https://arxiv.org/abs/2610.14709), v1, cs.CL, 10 Oct 2026. Code at github.com/AllSpark-Research/Vela. The data come from an unnamed "large social media platform".

## Summary

The paradigm is **user behaviour as supervision**. Each user's history is cut into segments of 72K–120K tokens. The first two-thirds is the visible history; what the user did next (views, searches, purchases, posts, follows, dislikes) is the hidden future and supplies the labels. There are 91 templates of two kinds:

- **Predictive** (objective choice or open prediction, scored by set-F1 against the future);
- **Rubric** (summaries, or explaining an observed future action, scored by task-specific rubrics).

From 100,000 users this yields 100,000 tasks. The composer and the scorer are both Qwen3.8-2.4T-A95B. A difficulty filter keeps tasks where the base model's mean score is between 0.2 and 0.8, leaving about 75,000. Training uses two corpora:

- **Vela-Instruct:** trajectories from the large Qwen3.8 teacher, used to fine-tune Qwen3.6-35B-A3B into **Vela-35B-A3B**;
- **Vela-RFT:** 9,375 of the base model's own best trajectories.

## Key points

- **VelaBench (982 held-out users):** 51.5 → **63.0 (+11.5)**. That beats DeepSeek-V4-Flash (58.6) and the Qwen3.8 teacher (62.3), but not Kimi-K3 (64.0).
- **User-related OOD:** +6.2 on average, with the largest gain on AdRecommendation (+11.2) and ProfileSummary +10.1.
- **General OOD:** +4.0 on average, from IFBench +6.7, CL-bench Life +5.6 and GPQA-Diamond +4.6.
- **Data scaling:** 4.7K examples already lift VelaBench to 59.2, rising to 63.0 at 75K. The general average goes from 55.8 to 58.2.
- **RL on top (GSPO, a [[grpo]]-family method):** starting from the Vela SFT checkpoint beats starting from the base by **7.7 points** on both VelaBench (66.2 vs 58.5) and ProfileSummary (37.6 vs 29.9).
- **Difficulty filtering:** +1.1 on user-OOD, no change in-domain.

## What it does not show

> [!warning] Newsletter vs paper
> The blurb's framing, "user behavior as supervision sparks LLM intelligence", repeats the title. The paper's own controls point to a confound. The general-capability gain appears only with **teacher-generated** trajectories. Vela-RFT, trained on the base model's own trajectories with the same behaviour labels, changes the general OOD average by **0.0** (54.2 → 54.2) and gains just 2.0 on VelaBench. So the +4 general gain is at least partly **[[knowledge-distillation|distillation]] from a 2.4T-parameter teacher**, not the behaviour signal alone. There is no ablation that distils the same teacher on non-behaviour data.

- **Self-overlapping judge.** The teacher also composes and scores the tasks, so VelaBench rewards matching its rubric.
- **Unnamed source.** The platform and the user-OOD benchmarks' provenance are not fully specified. Privacy handling is stated as removing identifiers plus an ethics review.

## Concepts & entities

- [[knowledge-distillation]] · [[supervised-fine-tuning]] · [[grpo]] · [[llm-as-a-judge]] · [[long-context-llm]] · [[qwen]]
- [[genrec-llm-recommendation-ranker-netflix-li-2026]] · [[kuaishou-explorer-llm-rec-challenge-cao-2026]]: other ways of turning behaviour logs into LLM training data. [[coral-llm-native-harness-production-recommender-azhar-2026]]
- [[openclaw]]: cited as the personal-assistant use case.
- [[generative-recommendation]]: concept page comparing LLM recommendation designs (Semantic IDs vs catalog scoring) and LLMs around recommenders.

## References

- [arXiv:2610.14709](https://arxiv.org/abs/2610.14709)
