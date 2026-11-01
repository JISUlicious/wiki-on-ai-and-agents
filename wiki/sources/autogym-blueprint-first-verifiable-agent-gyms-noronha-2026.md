---
title: "AutoGym: Blueprint-First Generation of Verifiable Agent Gyms"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - autogym-blueprint-first-verifiable-agent-gyms-noronha-2026.md
arxiv_id: "2609.22592"
authors:
  - Aarati Andrea Noronha
  - Kavya Ravikumar
  - Carly Xiaoyu Lin
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# AutoGym: Blueprint-First Generation of Verifiable Agent Gyms

**Aarati Andrea Noronha, Kavya Ravikumar, Carly Xiaoyu Lin** ([[amazon|Amazon AGI]]): [arXiv:2609.22592](https://arxiv.org/abs/2609.22592), v1, cs.AI, 18 Sep 2026. Preprint; no venue.

## Summary

An RL "gym" has three parts: a task, an executable environment and a [[verifier]]. AutoGym generates all three from a minimal domain seed or from prior model trajectories. The key move is to **reverse the usual generation order**. The system writes a *blueprint* first: the valid solution space, what the environment must contain and the verification criteria. Only then does it build ("materialize") the environment. Solvability becomes a construction requirement rather than something checked afterwards. The authors say this order was inspired by Agent World Model, which generates the task before it knows whether a valid solution path exists.

Difficulty is set by explicit generation parameters: task topology, interaction depth, capability axes, how obscure the question is, and the distractor budget. An **active curriculum** adjusts these parameters from failure analysis of a target model's rollouts. Ground truth comes from walking the solution path through the built environment. Verifiers score outcome correctness, process quality, resilience to distractors and (when specified) persona preferences, and an iterative repair loop drops or fixes inconsistent tasks.

## Key points

- **Setup.** Claude Sonnet 4.6 generates the gyms. Claude Opus 4.6 is the frontier reference and gpt-oss-120b the weaker comparison model. Each task gets 8 rollouts and is classed hard (mean score < 0.30), medium (0.30–0.80) or easy (≥ 0.80).
- **Difficulty is steerable (productivity gym, 7 tools, Opus 4.6).** Agent World Model (N=217) is 90% easy. The human expert-authored suite (N=300) is 21.5% hard. AutoGym C1 (low) is 10% hard, 34.7% medium and 55.3% easy. AutoGym C2 (mid-to-hard) is **39% hard**.
- **Repair removes fake difficulty.** Before repair, C2 was 57% hard. After three repair iterations, 280 of 350 tasks were kept (80%), and the hard share fell to 39% while easy rose from 30% to 51%. Many "hard" tasks had been broken tasks. C1 kept 297/350 (84.9%).
- **Separates models.** On C2, 61% of tasks are medium or easy for Opus 4.6 but only 30% for gpt-oss-120b (70% hard).
- **Failure-driven curriculum (temporal reasoning).** Generated from gpt-oss-120b failures; 92/100 tasks kept. It is 89.1% easy for Opus 4.6 but 41.4% hard or medium for gpt-oss-120b. The ordering is strictly one-sided: no task is harder for the stronger model.
- **Cost.** A six-domain suite of 600 tasks took under 10 hours, at about $100–200 per 50-task batch, with 86% kept after repair.
- **Quality.** Human inspectors validated every retained task. An LLM certification pass rated interface realism lowest (78.0/100) and environment realism at 86.0, the latter due to unnaturally integrated distractors.

## What it does not show

> [!warning] Newsletter vs paper
> The newsletter calls AutoGym a framework that "generates RL gyms", which suggests RL results. The only training evidence is Appendix G: GRPO on **one 8B model for 500 steps** on a separate 600-task instruction-following gym, reported as a rising *training-reward curve*. There is no held-out evaluation and no comparison with training on other gyms. The authors list full-scale training as future work.

- The main experiments measure **difficulty distribution and model separation**, not whether agents trained on these gyms improve.
- Only two evaluated models (Opus 4.6, gpt-oss-120b), and generation uses a single model family (Claude).
- Open-ended tasks still rely partly on LLM judging. The cloud-emulator substrate (App. E) is demonstrated but not evaluated.

## Concepts & entities

- [[agentic-rl]] · [[harnessed-agentic-rl]]: gyms are the scarce input. Compare the other environment-synthesis pipelines: [[agent-world-real-world-environment-synthesis-dong-2026]], [[skill2env-capability-oriented-environment-synthesis-allspark-2026]], [[envharness-awakening-static-worlds-huang-2026]], [[from-trainee-to-trainer-chen-2026]].
- Same batch, same "environment-first" thesis: [[autoenvscaling-data-flywheel-terminal-agents-yu-2026]] (the solver's rollouts drive the proposer) and [[vera-verifiable-environments-agentic-co-evolution-liu-2026]] (sandboxes built from trajectories).
- [[verifier]] · [[llm-as-a-judge]]: blueprint-defined ground truth replaces post-hoc judging wherever the task is deterministic.
- [[benchmark-contamination]]: the motivation for regenerable task sets.
- [[tau-bench]] · [[appworld]]: hand-authored environments cited as the realism standard.
- [[rl-environment-synthesis]]: concept page comparing automatic environment and gym generation pipelines.

## References

- [arXiv:2609.22592](https://arxiv.org/abs/2609.22592)
