---
title: "SkillGym: Internalizing Large-Scale Human Skills into LLMs for Real-World Problem Solving"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - skillgym-internalizing-human-skills-ge-2026.md
arxiv_id: "2609.27717"
authors:
  - Zhilong Ge
  - Yuting Shao
  - Yutao Yang
  - Yuxuan Cai
  - Jie Zhou
  - Kai Chen
  - Xin Li
  - Bo Zhang
  - Qin Chen
  - Liang He
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# SkillGym: Internalizing Large-Scale Human Skills into LLMs for Real-World Problem Solving

**Zhilong Ge, Yuting Shao, Yutao Yang, Yuxuan Cai, Jie Zhou, Kai Chen, Xin Li, Bo Zhang, Qin Chen, Liang He** (East China Normal University; Shanghai AI Laboratory): [arXiv:2609.27717](https://arxiv.org/abs/2609.27717), v2, cs.CL, 24 Sep 2026. Preprint; no venue.

## Summary

Human-written [[agent-skills]] (SKILL.md packages of workflow instructions, scripts and constraints) are normally loaded at inference time. SkillGym instead **turns them into training environments**, so the skill is learned into the weights. A skill-to-task pipeline pairs a reusable seed template for each sub-category with a target skill. It instantiates a concrete task, writes a code-based outcome checker and a reference solution, and runs **contrastive executions** (with and without the skill) to label whether the task actually depends on the skill. The result is 2,756 environments in 12 categories and 63 sub-categories, drawn from a skill registry that includes OpenClaw-style community skills.

Trajectories come from several teacher models (GPT-5.4, Nex-N2-Pro) under several harnesses ([[claude-code|Claude Code]], Codex). Only verified successes are kept: **8,364 of all sampled trials (17.4% success)**, averaging 49 tool calls and over 60k logged tokens. Qwen3.5-35B-A3B is then fine-tuned on them to give *SkillGym-Agent*.

## Key points

- **Under Claude Code, SFT on Qwen3.5-35B-A3B gives:**
  - GDPval-AA v2: **+199 Elo** (974 → 1173).
  - [[terminal-bench|Terminal-Bench]] 2.1: **+19.10 points** (39.33% → 58.43%).
  - SkillsBench v1.1: **+28.13** with skills (→ 51.47%) and **+14.68** without (→ 26.81%).
- **Codex harness.** Gains are larger in terminal execution (+35.96 on TB 2.1) and similar for skill-assisted solving (+27.69).
- **Against comparable-scale agents.** It beats Agents-A1, the strongest such baseline, by 189 Elo, 14.61 points on TB 2.1, and 21.18 / 13.44 points on SkillsBench with / without skills.
- **Against reported frontier scores.** It roughly ties Claude Sonnet 4.6 on TB 2.1 (58.43 vs 58.5). On skill-assisted SkillsBench (51.47) it beats Claude Sonnet 4.6 (47.2), GPT-5.4 Mini (41.4) and DeepSeek V4 Pro Preview (50.1).
- **Teacher pooling helps.** Combining GPT-5.4 and Nex-N2-Pro trajectories beats the best single teacher. Cross-harness pooling lifts Codex further, by up to +13.11 on skill-assisted SkillsBench.
- **Skills stay complementary.** Supplying skills at inference still improves the fine-tuned agent, so internalizing them does not make them redundant.

## What it does not show

> [!warning] Newsletter vs paper
> The newsletter groups SkillGym with RL gyms, but **every reported gain comes from supervised fine-tuning on distilled teacher trajectories**. The environments "support" outcome-reward RL, but no RL was run; the conclusion lists it as future work. The newsletter's numbers (+19.10, +28.13, 51.47%) match the paper.

- No decontamination between skill-derived training tasks and SkillsBench or Terminal-Bench is reported. Because SkillsBench is itself built around skills, overlap in skill sources is a live concern.
- The frontier comparisons use *reported* public scores under other harnesses, not reruns.
- Gains combine distillation from stronger teachers with skill content. There is no matched SFT baseline on an equal number of non-skill-derived trajectories.

## Concepts & entities

- [[agent-skills]] · [[skill-md-format]] · [[skill-acquisition]]: moving skills from the context window into the weights. Contrast with inference-time skill work such as [[skill2env-capability-oriented-environment-synthesis-allspark-2026]] (skills → environments for general agents, the closest prior) and [[skill-zero-lu-2026]].
- [[supervised-fine-tuning]] · [[knowledge-distillation]]: the training method actually used.
- [[terminal-bench]] · [[claude-code]]: evaluation benchmark and harness.
- [[procedural-memory]]: "reusable procedural competence" is the stated target.
- Same batch: [[autoenvscaling-data-flywheel-terminal-agents-yu-2026]], which reports Terminal-Bench 2.1 RL gains on the same Qwen3.5-35B-A3B backbone.
- [[rl-environment-synthesis]]: concept page comparing automatic environment and gym generation pipelines.

## References

- [arXiv:2609.27717](https://arxiv.org/abs/2609.27717) · GitHub: ECNU-ICALK/SkillGym
