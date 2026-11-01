---
title: "ReHarness: Recurrent Harness for Agentic Continual Learning"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - reharness-recurrent-harness-agentic-continual-learning-xiao-2026.md
arxiv_id: "2610.12668"
authors:
  - Yuxin Xiao
  - Zhiyun Lu
  - Jianyu Wang
  - Yangtian Zhang
  - Yang Song
  - Qifan Wang
  - Bailin Wang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# ReHarness: Recurrent Harness for Agentic Continual Learning

**Yuxin Xiao, Zhiyun Lu, Jianyu Wang, Yangtian Zhang, Yang Song, Qifan Wang, Bailin Wang** ([[mit|MIT]]; [[meta|Meta]] MRS Research; Yale) — [arXiv:2610.12668](https://arxiv.org/abs/2610.12668), v1, cs.LG, 8 Oct 2026. Preprint. The work was done during Xiao's Meta internship.

## Summary

Memory and reflection harnesses usually update their state only **forward**: they extract, store and retrieve. Once written, a reflection is rarely revised based on how it worked later. ReHarness adds a **backward revision** step under a strictly online stream, with no replay, no validation set and no repeated trials.

After each instance, a revision agent revisits the *k* = 5 reflections that were retrieved for that instance. It uses the new trajectory as feedback and asks what those reflections should have said to better inform it. Any new content must stay grounded in the reflections' **own source trajectories**. The revised state is carried forward, which closes a recurrent loop of forward read and write plus backward revision.

## Key points

- **Setup.** Three continual-learning benchmarks cover 16 tasks: CL-Bench (6), MemoryBench (5 English writing tasks) and MemoryArena (5). Three backbones are used: Claude Opus 4.8, GPT-5.5 and Gemini 3.7 Flash, all at high thinking effort. Each experiment has 3 sampled stream rollouts.
- **Metric.** Normalized gain over the stateless backbone: (r − r_stateless)/(r_max − r_stateless).
- **Overall mean normalized gain** across the nine benchmark–backbone cells: ReHarness **0.240**. The baselines score Top-K Raw 0.168, Full Raw 0.164, Mem0 0.131, GEPA-Online 0.092, DoT 0.071, ACE 0.043 and ReasoningBank 0.034.
- ReHarness ranks first or tied in **8 of 9** cells; it ties Full Raw on MemoryArena with GPT. It beats every baseline on paired Wilcoxon tests with Bonferroni correction. The margins over the raw-context baselines are narrow but significant (p_adj = 0.039 vs Top-K, 0.043 vs Full Raw).
- **Raw context is the baseline to beat.** Consistent with CL-Bench's authors, keeping raw trajectories beats most structured memory and reflection methods. Reflection methods (ReasoningBank, DoT, ACE) sometimes go negative.
- **Revisions are grounded.** Over 95% of revised claims are entailed by their sources, with leakage of 0.8–2.1%.
- **Token use** (CL-Bench, Claude). ReHarness uses about 41.6M tokens, Top-K Raw 52.7M and Full Raw 142.1M, so Full Raw needs **3.4×** as many.
- In ablation, removing the revised reflections as demonstrations lowers CL-Bench/Claude gain from 0.217 to 0.191.

> [!warning] Newsletter vs paper
> The newsletter blurb (MIT/Meta/Yale; streaming with no replay; 3 benchmarks, 16 tasks, 3 backbones) is accurate.

## What it does not show

- **Weak where rewards are noisy.** Gains are marginal on exploitable poker, the stochastic CL-Bench task; the authors flag this.
- **Raw trajectories are still stored** for later revision, which adds storage overhead.
- **k was chosen after the fact.** The sweep is post hoc, with gains declining for k > 20.
- **Modest absolute gains.** Normalized gains of 0.1–0.4 show that most of the headroom over the stateless agent remains.

## Concepts & entities

- [[continual-learning]] · [[catastrophic-remembering]] — the stability–plasticity decomposition (ReHarness is highest on both).
- [[reflection-mechanism]] · [[self-reflection]] · [[memory-evolution]] · [[memory-consolidation]] — backward revision as feedback-driven reflection editing.
- [[agentic-context-engineering]] · [[mem0]] · [[gepa]] — baselines.
- [[harness-continual-learning-kang-2026]] — related continual-learning harness, which uses extra data that a strict stream forbids.
- [[agentic-harness-engineering]] · [[long-term-memory]] · [[in-context-learning]]

## References

- [arXiv:2610.12668](https://arxiv.org/abs/2610.12668)
