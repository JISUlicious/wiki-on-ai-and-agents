---
title: "AutoEnvScaling: Automating the Data Flywheel with Terminal Agents"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - autoenvscaling-data-flywheel-terminal-agents-yu-2026.md
arxiv_id: "2610.14163"
authors:
  - Simon Yu
  - Baolin Peng
  - Bo Liu
  - Zhengyan Shi
  - Zhaoyang Wang
  - Hao Cheng
  - Qianhui Wu
  - Wenlin Yao
  - Natasha Jaques
  - Weiyan Shi
  - Jianfeng Gao
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# AutoEnvScaling: Automating the Data Flywheel with Terminal Agents

**Simon Yu, Baolin Peng, Bo Liu, Zhengyan Shi, Zhaoyang Wang, Hao Cheng, Qianhui Wu, Wenlin Yao, Natasha Jaques, Weiyan Shi, Jianfeng Gao** ([[microsoft-research|Microsoft Research]]; Northeastern; [[stanford-university|Stanford]]; [[university-of-washington|UW]]; UNC). The first author did this work as an intern at MSR. [arXiv:2610.14163](https://arxiv.org/abs/2610.14163), v1, cs.SE, 9 Oct 2026. Preprint; no venue.

## Summary

The paper treats **environment design as a terminal task**. A *proposer* agent works in a sandbox with a coding harness, web access, the solver's past rollouts and child sandboxes. It writes Harbor-format RL environments (instruction, Docker setup, tests, reference solution). Each environment must pass a *validation* check (it builds, the reference passes, an empty solution fails) and a *calibration* check (the current solver's success rate falls in a target band). Failures go back to the proposer. The solver trains on accepted environments, and its new rollouts guide the next round. When one model plays both roles, both are trained with role-specific group baselines. The proposer reward is −0.25 when validation passes but calibration fails, and lower when validation fails.

## Key points

- **Generation quality.** Across eight proposer models, an agentic proposer beats Prompt-only and Prompt+Env-Feedback on valid-environment rate. The largest jump is DeepSeek-V4-Flash, **3.5% → 89.0%**: this is the "up to 25×". Weaker models no longer fall behind (Sol/Terra/Luna). Cost per valid environment drops for 6 of 8 models, by **up to 75%**.
- **Solver RL vs a matched static pool.** Tmax is regenerated with the *same* proposer and trained for matched steps. Results are held-out Terminal-Bench 2.1, avg@5:
  - Qwen3.5-9B: 19.1 base → Tmax 27.2 / 29.2 → **AutoEnvScaling 33.5 / 36.6** (GPT-5.6-sol / Claude Opus 5 proposer), i.e. +6.3 / +7.4.
  - Qwen3.5-35B-A3B: 26.7 → Tmax 34.6 / 35.5 → **38.9 / 39.3**, i.e. +4.3 / +3.8.
- **Solver feedback matters.** Remove past solver trajectories from the proposer's workspace and the 35B-A3B score falls 3.6 / 6.7 points, ending 0.7 above Tmax (GPT proposer) and 2.9 *below* it (Claude proposer).
- **Self-improvement (one model in both roles).** Qwen3.6-35B-A3B starts at a Cold-Start checkpoint. TB 2.1 avg@5 goes **41.1 → 53.3 (+12.2)**, which is **+6.1 over Tmax** from the same checkpoint. Terminal-Bench 4.0 goes 1.6 → 3.2 (Tmax: +1.4). Without proposer-side training the run collapses as validation errors grow.
- **Training efficiency.** The usable-group rate (groups with mixed outcomes) is 3.1× higher than Tmax, and rollout collection per step takes 42% less time.
- The proposer also evolves its own tools between rounds, for example caching useful GitHub tasks and Docker images as a skill.

## What it does not show

> [!warning] Newsletter vs paper
> The newsletter's "Qwen3.6-35B-A3B TB2.1 41.1 → 53.3" is correct but is a gain over the **Cold-Start** checkpoint. That checkpoint was itself SFT'd on **600 proposer trajectories from Claude Opus 5**, because the base model produced valid environments only 32.3% of the time. The like-for-like gain over the static pool is +6.1. "Self-improvement" here starts from frontier distillation.

- Passing validation (reference passes, empty solution fails) does not prove an environment is correct, as the authors note.
- Cross-domain environments (SWE, computer use, professional work, GPU kernels) are generated and validated, but no training gains are shown in those domains.
- Decontamination is a 13-gram textual-overlap check against Terminal-Bench, so paraphrased leaks are not ruled out. TB 4.0 scores stay very low (3.2).

## Concepts & entities

- [[terminal-bench]] · [[agentic-rl]] · [[harnessed-agentic-rl]] · [[grpo]].
- [[self-improving-agent]]: proposer/solver co-training as a recursive self-improvement step. Compare [[from-trainee-to-trainer-chen-2026]] (the policy as environment engineer) and [[frognano-training-a-4b-coding-agent-via-online-task-synthesis-kim-2026]] (online task synthesis).
- [[agentic-harness-engineering]]: the authors read the 3.5% → 89% jump as support for the "mismanaged geniuses" view that scaffolds hold models back. See [[harness-vs-model-attribution]].
- [[benchmark-contamination]] · [[verifier]].
- Same batch: [[autogym-blueprint-first-verifiable-agent-gyms-noronha-2026]] (curriculum from failure analysis), [[vera-verifiable-environments-agentic-co-evolution-liu-2026]], [[skillgym-internalizing-human-skills-ge-2026]] (SFT gains on TB 2.1 with the same backbone).
- [[microsoft-research]] · [[qwen]].
- [[rl-environment-synthesis]]: concept page comparing automatic environment and gym generation pipelines.

## References

- [arXiv:2610.14163](https://arxiv.org/abs/2610.14163)
