---
title: "VERA: Scaling Verifiable Environments for Agentic co-Evolution"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - vera-verifiable-environments-agentic-co-evolution-liu-2026.md
arxiv_id: "2610.05923"
authors:
  - Junqi Liu
  - Yongyang Pan
  - Zhuosong Jiang
  - Dongbai Li
  - Bo Zhang
  - Xitong Ling
  - Sheng Wang
  - Hanrong Ye
  - Yufan He
  - Can Zhao
  - Pengfei Guo
  - Dong Yang
  - Andriy Myronenko
  - Yuyin Zhou
  - Tianyu Liu
  - Daguang Xu
  - Yucheng Tang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# VERA: Scaling Verifiable Environments for Agentic co-Evolution

**Junqi Liu, … Daguang Xu, Yucheng Tang** (17 authors; project lead Yucheng Tang, [[nvidia|NVIDIA]]; with UC Santa Cruz, [[uiuc|UIUC]], NUS, [[tsinghua-university|Tsinghua]], Xi'an Jiaotong, UPenn): [arXiv:2610.05923](https://arxiv.org/abs/2610.05923), v1, cs.AI, 5 Oct 2026. Preprint; no venue.

## Summary

VERA argues that long-horizon research agents (for example, medical-image research pipelines) are held back by environments, not algorithms: existing environments score only the final outcome. Its **Auto-Rubric** pipeline turns benchmark trajectories (from [[swe-bench]], [[mle-bench]] and AutoMedBench, with evaluation instances excluded) into restartable sandboxes. A coding agent writes binary rubrics covering action arguments, execution effects, artifacts and outputs. *Stage-wise* sandboxes can restore the workspace at the start of any of five stages: plan, setup, verify, execute and submit. Executable checks and an agent judge inspect each sandbox, and a failed executable check overrides the judge. Only sandboxes that pass enter the bank, which holds 9,000+ environments and is open-sourced.

On top of the bank, VERA runs **model–harness co-evolution** in alternating rounds, with the other component frozen each time. An LLM attribution step reads stage-wise scores and traces to decide where the agent fails. Model rounds run full-parameter [[grpo|GRPO]] with rubric rewards that a Reward Verifier checks against rollout evidence. Harness rounds add, merge or remove skills, and an edit is kept only if paired reruns with the frozen model show improvement on a development set. A rejected round falls back to the last accepted checkpoint.

## Key points

- **9B (Qwen3.5-9B), vs single-axis methods** (ADAS harness search, GRPO, RaR, COS-PLAY):
  - AutoCoWorkBench: 31.0 Overall vs 20.7 for ADAS, the strongest baseline (**+10.3**).
  - AutoMedBench: 69.1 vs 56.1 for VERA's own *harness-only* variant (**+13.0**).
  - It also leads on SWE-Bench Verified (54.2 Pass@1) and MedXpertQA Text (50.8), which use official scoring that differs from the training rubrics.
- **Ablation: each axis supplies about half the gain.** AutoMedBench: LLM-only 43.3, harness-only 56.1, full loop 69.1. AutoCoWorkBench: 18.9 / 18.2 / 31.0. On Med, the harness arm carries more of the gain.
- **27B (Qwen3.8-27B), with benchmark-default runners.** AutoCoWorkBench 71.6, **1.0 below GPT-5.6-Sol**. AutoMedBench 80.7, **1.2 below Claude Opus 4.8** (81.9). Claude drops to 80.5 inside the productivity harness, so rankings flip with the harness.
- **Out-of-distribution.** Automation-Bench (Zapier, public 600): 43.8% pass rate, ranked 2nd. AgentClinic: 76.4%, ranked 3rd. These are +19.2 and +32.5 points over the base agent.
- **General capability depends on scale.** At 4B and 9B the agents lose **56.4–59.7** and **15.8–55.8** AIME points respectively. At 27B the CoWork agent improves on AIME 2026, [[alfworld|ALFWorld]], [[gpqa|GPQA-Diamond]] and IF-Bench. The Med agent loses 4.0 on ALFWorld and 0.7 on IF-Bench.
- **Cost.** One 27B co-evolution run takes about 5,000–8,000 GPU-hours.

## What it does not show

> [!warning] Newsletter vs paper
> - The abstract says VERA-27B "surpasses the baseline" at 71.6 / 80.7, and the newsletter repeats both numbers as wins. Section 4.2 and the Table 3 caption put VERA-27B **below** the best frontier system in both domains (−1.0 vs GPT-5.6-Sol, −1.2 vs Claude Opus 4.8). The intro's claim that it surpasses Claude Opus 4.8 on AutoCoWorkBench holds only against Claude, not against the best baseline.
> - The "+13.0" on Med is measured against VERA's own harness-only ablation, not an external method.

- AutoMedBench comes from overlapping authors (Liu et al. 2026b), and AutoCoWorkBench is new. Both headline benchmarks are in-house.
- Judging, verification and attribution all use GPT-5.6-Sol, and judge sensitivity is untested. Full baseline comparisons exist only at 9B. Environment count is never varied, so "scaling" environments is not itself measured.
- The 4B and 9B results show heavy forgetting of math (AIME), which the summary framing ("retains general capabilities") mentions only for 27B.

## Concepts & entities

- [[self-improving-agent]] · [[agentic-rl]] · [[harnessed-agentic-rl]]: VERA alternates weight updates with harness edits, each gated on a development set. Related co-evolution work: [[co-evolving-decision-and-skill-bank-agents-wu-2026]].
- [[harness-vs-model-attribution]]: the ablation is a direct model-vs-harness split, and the Claude 80.5 vs 81.9 runner gap is another data point.
- [[agentic-harness-engineering]] · [[adas]] · [[skill-library]]: the harness arm edits skills.
- [[verifier]] · [[llm-as-a-judge]] · [[outcome-based-reward]]: rubric rewards are checked against evidence instead of using outcome-only tests.
- Same batch: [[autogym-blueprint-first-verifiable-agent-gyms-noronha-2026]], [[autoenvscaling-data-flywheel-terminal-agents-yu-2026]].
- [[qwen]] · [[nvidia]].
- [[rl-environment-synthesis]]: concept page comparing automatic environment and gym generation pipelines.

## References

- [arXiv:2610.05923](https://arxiv.org/abs/2610.05923)
