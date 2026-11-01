---
title: "Terminal-RSI: Recursive Self-Improving Terminal Agents via Recursive Self-Rewrite and Policy-Refine"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - terminal-rsi-recursive-self-improving-terminal-agents-yang-2026.md
arxiv_id: "2610.22192"
authors:
  - Junyao Yang
  - Zongxia Li
  - Yucheng Shi
  - Zhongzhi Li
  - Ruhan Wang
  - Ruiwen Zhou
  - Haitao Mi
  - Shuicheng Yan
  - Leowei Liang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Terminal-RSI: Recursive Self-Improving Terminal Agents

**Junyao Yang, Zongxia Li, Yucheng Shi, Zhongzhi Li, Ruhan Wang, Ruiwen Zhou, Haitao Mi, Shuicheng Yan, Leowei Liang** (Tencent Hy Frontier · National University of Singapore · UMD · University of Georgia · Indiana University) — [arXiv:2610.22192](https://arxiv.org/abs/2610.22192), v1, Oct 2026. Preprint.

## Summary

Terminal-RSI is a five-stage post-training recipe for one terminal agent. It starts from Qwen3.8-27B and alternates two phases: **Recursive Self-Rewrite (RSR)** and **Recursive Policy-Refine (RPR)**. RSR is actor-only [[supervised-fine-tuning|SFT]] on the agent's own *verified* trajectories. Successful runs under complementary harnesses are rewritten by a planner, checked by a leakage reviewer, and re-executed fresh under the target harness. RPR is [[ppo|PPO]] whose critic is **inherited** across stages rather than reset: a warm-up critic model plus phase-local value calibration after each actor change. The paper also contributes:

- **TB-GAE.** Turn-boundary skip-observation GAE, which keeps per-token baselines but applies decay across interaction turns.
- **FDR.** Family Dense Reward, which calibrates partial-progress credit across the 46 task families with different numbers of checks.
- **SPR.** Success-Prioritized Reward, which keeps partial credit below strict completion.

The sequence is Base → RSR-1 → RPR-1 → RSR-2 → RPR-2 (= Terminal-RSI-27B).

## Key points

- **Final checkpoint (Mean@3):** Terminal-Bench 4.0 **19.84%** (Base 0.00), TB 2.1 **86.52%** (51.70), TB 3.0 **16.95%** (0.00), TB Hard **79.00%** (31.00). The four-suite macro mean goes 20.68 → 34.25 → 44.50 → 48.75 → **50.58%**.
- **Long-horizon:** LHTB process reward 0.21 → 0.37, strict completions 0/46 → 5/46. SWR100 3% → 6%, flat since RPR-1.
- **Both phases contribute.** RPR-1 adds 7.43 / 7.49 / 10.57 / 15.50 pp over RSR-1 on TB 2.1 / 3.0 / 4.0 / Hard. Verified rewriting as an "exploration anchor" lets RPR-1 beat direct PPO from the base model by up to +29.2 on TB 2.1 at step 20. Direct PPO left TB 2.1 *below* base (48.31).
- **Reward rescoring (offline).** On the same 4,352 fixed trajectories, failures' share of total reward mass falls from **50.09% (FDR) to 28.72% (SPR)**. Failures scoring ≥1 drop from 158 to 0.
- **Reference comparison.** TB 2.1 86.52% sits above the officially listed GPT-6 Sol (85.40) and Kimi K3 (85.00). GPT-6 Sol is higher on TB 4.0. These are quoted numbers, not re-runs.

> [!warning] Newsletter vs paper
> "Dense reward failed runs 50.09% → 28.72%" is an **offline rescoring of fixed trajectories**. It shows how SPR reallocates credit, not that training under SPR reduced failures; the paper says so explicitly. "TB avg 20.68 → 50.58" is the macro-mean over four Terminal-Bench suites. Tencent is the lead lab, with NUS as the corresponding author's affiliation.

## What it does not show

- **No causal attribution.** The authors list compute-matched alternation, isolated TB-GAE, matched online reward, and inherited-vs-reset critic controls as *still needed*. Learning rates, rewards and initialization change together across stages.
- **One seed per stage.** Three evaluation runs, no independent training seeds, and checkpoint selection without a protected validation split.
- **Second cycle gains are thin.** TB 2.1 is unchanged from RSR-2 to RPR-2, SWR100 plateaus, and the paper calls it "a plateau rather than a further second-cycle gain".
- **"Recursive" means staged self-distillation plus RL on one backbone.** The agent does not modify its own learning procedure.

## Concepts & entities

- [[self-improving-agent]] · [[agentic-rl]] · [[harnessed-agentic-rl]] — RL on terminal agents with critic handoff.
- [[terminal-bench]] · [[terminal-bench-benchmarking-agents-cli-merrill-2026]] — benchmark family (2.1, 3.0, 4.0, Hard).
- [[ppo]] · [[supervised-fine-tuning]] · [[outcome-based-reward]] · [[reward-hacking]] — SPR is a guard against partial-credit farming.
- [[qwen]] — backbone.
- Siblings: [[recursive-self-improvement-multi-agent-self-supervision-lee-2026]] (MASS, non-verifiable RSI), [[agentopsd-recursive-self-distillation-agentic-rl-wang-2026]], [[rethinking-evaluation-harness-evolution-wang-2026]] (TB 2.1 harness-evolution negative result).
- [[recursive-self-improvement]]: concept page for RSI definitions, autonomy levels and recurring caveats across papers.

## References

- [arXiv:2610.22192](https://arxiv.org/abs/2610.22192)
