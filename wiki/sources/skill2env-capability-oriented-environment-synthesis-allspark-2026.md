---
title: "Skill2Env: Capability-Oriented Environment Synthesis from Skills for General Agents"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - skill2env-capability-oriented-environment-synthesis-allspark-2026.md
arxiv_id: "2609.33772"
authors:
  - AllSpark Team
  - Weiyi Xu
  - Xiaowen Yang
  - Wen Da
  - Hang Xu
  - Canwei Li
  - Hongjie You
  - Pusen Dong
  - Yucheng Zeng
  - Zhaokai Luo
  - Mu Chuan
first_author: AllSpark Team
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Skill2Env: Capability-Oriented Environment Synthesis from Skills for General Agents

**AllSpark Team.** Contributors in order: Weiyi Xu\*, Xiaowen Yang\*, Wen Da (project lead), Hang Xu, Canwei Li, Hongjie You, Pusen Dong, Yucheng Zeng, Zhaokai Luo, Mu Chuan (corresponding). The paper gives no institutional affiliation. [arXiv:2609.33772](https://arxiv.org/abs/2609.33772) (v2), cs.AI, 29 Sep 2026. Preprint.

## Summary

[[agent-skills|Skills]] say *how* to do something but not *what task* to train on. Skill2Env turns a skill folder into an executable training task by organising synthesis around **five capability demands**: environment understanding, planning, skill usage, long-horizon consistency and error recovery. These are expressed through a pool of **100 reusable difficulty patterns**, for example *distributed evidence*, *implicit constraints*, *state dependencies*, *resource budgets* and *schema mismatches*. For each skill, a synthesis agent (Kimi-K3) picks compatible patterns and writes a **task blueprint**, an "author-side contract" listing objective, challenges, environment facts, *information boundaries* (what is given, discovered or inferred) and acceptance criteria. From the blueprint it builds the instruction, the Docker execution substrate, the workspace and a rubric evaluator that mixes deterministic checks with an LLM judge (Qwen3.5-397B). **Iterative Task Hardening** re-runs a fixed solver (Qwen3.6-35B-A3B). Any task scoring above 0.7 is diagnosed for shortcuts or under-used patterns and revised; new challenge types found this way are added to the pattern pool.

Source pool: more than 47K public skill folders from ClawHub ([[openclaw]]'s registry), screened for utility, Linux compatibility and setup feasibility down to over 3K. The pipeline yields **2,963 executable tasks**.

## Key points

- **Main result (Table 1).** SFT of Qwen3.6-35B-A3B on **1.5K** DeepSeek-V4-Flash trajectories with r > 0.9, run for 5 epochs, raises the seven-benchmark average from **36.6 to 45.0 (+8.4)**. [[terminal-bench|Terminal-Bench 2.1]] goes 44.9 → 58.4 (+13.5) and SkillsBench 32.5 → 46.9 (+14.3). The other gains are SWE-bench Multilingual +7.7, AutomationBench +7.3, VitaBench +5.9, Claw-Eval +5.5 and τ³-Banking +4.5. The result beats Qwen3.5-397B-A17B on 6 of 7 benchmarks.
- **Across harnesses (Fig. 4).** Terminal-Bench 2.1 improves under all four harnesses: pi 46.1 → 57.3, Terminus-2 44.9 → 58.4, OpenCode 38.2 → 55.1, Claude Code 40.5 → 52.8.
- **SkillsBench with skills on vs off.** The gain is +14.3 with skill auto-loading and +10.4 without it. With skills on, 7 of 8 domains improve (finance unchanged), led by Media (+46.7 on only 5 tasks).
- **Hardening works (Fig. 5; 500 paired tasks, DeepSeek-V4-Flash under pi).** Full-credit rate falls from 48.4% to 24.8% to 15.4% across iterations 0–2, while mean turns rise from 25.9 to 36.5 to 38.9. At a fixed 500 trajectories per stage, the downstream average rises from 40.58 to 42.51 to 42.87.

## Concepts & entities

- [[agent-skills]] · [[skill-md-format]] — the input unit is a public skill folder. [[agentskills-io-spec-2025]] is the format the ClawHub skills follow.
- [[terminal-bench]] · [[terminal-bench-benchmarking-agents-cli-merrill-2026]] — the headline benchmark.
- [[agent-world-real-world-environment-synthesis-dong-2026]] — a prior environment-synthesis approach, from real-world environments rather than skills.
- [[knowledge-distillation]] · [[supervised-fine-tuning]] — the training signal is SFT on teacher trajectories.
- [[llm-as-a-judge]] — the rubric items that are not programmatically checkable.
- [[kimi-k3-moonshot-2026]] (synthesizer) · [[glm-5-2]] (baseline) · [[openclaw]].
- [[skill-evaluation]] — SkillsBench measures skill use directly.

## What it does not show

- **There is no distillation control.** The gain could come from 1.5K trajectories of a much stronger teacher (DeepSeek-V4-Flash averages 54.4, and the student reaches 45.0) rather than from capability-oriented synthesis. No arm trains on the same number of teacher trajectories from unhardened or pattern-free tasks. The FACET comparison is a single source-reported Terminal-Bench number on a different backbone (Qwen3.5-27B).
- No RL is run, although the rubric rewards would allow it.
- Each benchmark is run once, with no variance. The hardening-utility gain (+2.29) comes from one random sample of 500 per stage.
- Pattern selection and blueprints are LLM-authored. Rubric-judge agreement with humans is not measured.

## References

- [arXiv:2609.33772](https://arxiv.org/abs/2609.33772)
