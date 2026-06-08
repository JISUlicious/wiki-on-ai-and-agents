---
title: "A Primer in Post-Training Reasoning Data: What We Know About How It Works"
type: source
created: 2026-06-08
updated: 2026-06-08
sources:
  - primer-in-post-training-reasoning-data-li-2026.md
arxiv_id: "2606.02113"
authors:
  - Yaoming Li
  - Guangxiang Zhao
  - et al.
first_author: Yaoming Li
year: 2026
tags: [2026]
status: complete
importance: medium
---

# A Primer in Post-Training Reasoning Data: What We Know About How It Works

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-05-31–06-07).

## Summary

Post-training has become a primary driver of progress in large reasoning models, and the reasoning data used in that stage is often the key variable for success. But the literature is scattered across dataset papers, RL recipes, reward-model studies, benchmarks, and frontier system reports. This is the **first primer to synthesize over 150 public studies and system reports** on post-training reasoning data, organized around four questions: what data objects exist, what makes them useful, how they are constructed, and how their gains scale.

The central reframing is that a reasoning-data item is not just a prompt–response pair but a **feedback-bearing record** linking task context/state, model behavior, judging feedback, and attribution metadata. This framing exposes common misleading assumptions — long chain-of-thought traces are not necessarily faithful; "harder" data is useful only relative to a base model and verifier; large corpora can still have poor coverage under uncontrolled mixture/leakage/lineage; and in agentic settings, over-cleaned success trajectories strip out the failures, retries, and state changes needed for credit assignment. The result is offered as an attribution framework for inspecting future reasoning-data releases and post-training recipes.

## Key points

- First survey to consolidate 150+ studies/system reports on post-training reasoning data into a common language.
- Organizes the field around four questions: what data objects exist, what makes them useful, how they are built, and how they scale.
- Reframes a reasoning-data item as packaging: problem/state + model behavior + judging feedback + attribution metadata.
- Debunks intuitive-but-misleading assumptions (long CoT ≠ faithful; harder data is base/verifier-relative; bigger ≠ better coverage).
- Notes agentic pitfall: over-cleaned success trajectories remove the failures/retries needed for credit assignment.
- Ships a companion repo (Awesome-LLM-Reasoning-Data); authors at Peking University, Tsinghua, and Qiyuan Tech.

## Concepts & entities

- [[chain-of-thought]] — CoT traces as a core reasoning-data object; faithfulness caveat.
- [[reinforcement-learning]] — RL recipes and reward models as part of the post-training stage.
- [[scaling-laws]] — how reasoning-data gains scale.

## References

- [arXiv:2606.02113](https://arxiv.org/abs/2606.02113)
