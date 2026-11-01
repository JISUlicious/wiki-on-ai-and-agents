---
title: "Critical-State RL: Diagnosing Trainable States for Multi-Turn Tool Use"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - critical-state-rl-trainable-states-tool-use-chen-2026.md
arxiv_id: "2609.24985"
authors:
  - Zixiang Chen
  - Wenting Zhao
  - Zhepeng Cen
  - Akshara Prabhakar
  - Jielin Qiu
  - Jianguo Zhang
  - Zhiwei Liu
  - Tulika Manoj Awalgaonkar
  - Liangwei Yang
  - Shelby Heinecke
  - Silvio Savarese
  - Huan Wang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Critical-State RL: Diagnosing Trainable States for Multi-Turn Tool Use

**Zixiang Chen, Wenting Zhao, Zhepeng Cen, Akshara Prabhakar, Jielin Qiu, Jianguo Zhang, Zhiwei Liu, Tulika Manoj Awalgaonkar, Liangwei Yang, Shelby Heinecke, Silvio Savarese, Huan Wang** (Salesforce AI Research): [arXiv:2609.24985](https://arxiv.org/abs/2609.24985), v1, cs.LG, 21 Sep 2026. Preprint; no venue.

## Summary

A multi-turn tool-use failure often hinges on one model call, but trajectory-level reward does not say *which* call. When rewards depend on later turns, a mix of successes and failures in a [[grpo|GRPO]] group can come from **continuation noise** rather than from differences between the current actions. Critical-State RL diagnoses this *before* training. Given candidate calls (defined by the task, and proposed by humans in some applications) and a local reward, it uses **nested sampling**: sample alternative actions at a fixed context, then hold each action fixed and resample its continuation. This splits reward variance into an action-dependent part and noise. The diagnostic also checks that the reward reflects task success and that the reference policy leaves room to improve. The best-qualifying call is then trained alone with **contextual-bandit** updates. Surrounding calls supply context or reward but receive no gradient.

The theory links action-dependent variance to the improvement possible under a small KL budget. It bounds finite-sample ranking error and gives a sufficient condition for local updates to improve full-rollout return.

## Key points

- **Four-cell study.** Gemma-4-26B-A4B, no-think mode, on BFCL v4 multi_turn, with paired deterministic evaluation on 200 items:
  - miss_func: the diagnostic picks the *recovery* turn (after the tool appears). Score goes **0.14 → 0.283 ± 0.015 (+14.3 pp, 4 seeds)**. Training the alternative *decision* turn instead gives 0.095 (−4.5 pp).
  - miss_param: the diagnostic picks the *decision* turn (before the argument arrives). Score goes 0.435 → 0.473 ± 0.010 (+3.8 pp). Training the alternative gives +1 pp. The authors call this result "directional".
  - Each fixed rule (always-decision or always-recovery) helps one category and hurts or stalls the other.
- **Wording transfer.** With an unseen bridge wording that names no tools, the recovery gain holds (0.095 → 0.225, +13.0 pp).
- **Versus full multi-turn training.** Monte-Carlo return-to-go reaches 0.997 on gym validation but only 0.45 on BFCL miss_param, versus 0.473 for Critical-State RL. That comparison is single-seed.
- **Applications.** These use the same workflow but are not matched tests:
  - Logged repeat-call avoidance (Nemotron-Super-120B repeats a call on 30% of flagged steps; GPT-4.1 on 0.61%): call-vs-answer agreement with GPT-4.1 goes 37% → about 75%.
  - Nemotron miss_func decision training: +4.4 pt.
  - BFCL memory: the xLAM sub-task goes 34.54% → 50.54%, and Gemma 38.1% → 52.7%.

## What it does not show

> [!warning] Newsletter vs paper
> "+~14 points on BFCL v4 missing-function" is accurate. It applies to **one model** (Gemma-4-26B-A4B, no-think) on one sub-category, from a low base (14% → 28%). Nemotron *also* benefits, but there the diagnostic picks a different turn and the gain is +4.4.

- The diagnostic chooses between categories of turns. Variance thresholds and magnitudes are not calibrated.
- Candidate calls must be specified in advance, and some applications use human-proposed candidates plus an LLM judge to filter rows.
- The repeat-call, Nemotron and memory settings are demonstrations, not controlled tests of the diagnostic.

## Concepts & entities

- [[agentic-rl]] · [[grpo]] · [[reinforcement-learning]]: turn-level credit assignment. Compare [[agentopsd-recursive-self-distillation-agentic-rl-wang-2026]] (a turn-level advantage multiplier).
- [[function-calling]] · [[tool-use]]: BFCL multi-turn missing-function and missing-parameter tasks.
- [[outcome-based-reward]] · [[process-reward-model]]: local labels sit between trajectory-level outcome reward and per-step process reward.
- [[memory-management]]: BFCL memory sub-task applications.
- Same batch: [[sharpening-tax-in-post-training-oh-2026]] (also BFCL v4 multi-turn), [[mimesis-user-simulators-training-environments-phan-2026]].

## References

- [arXiv:2609.24985](https://arxiv.org/abs/2609.24985)
