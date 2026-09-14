---
title: "FrogNano: Training a 4B Coding Agent via Online Task Synthesis"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - frognano-training-a-4b-coding-agent-via-online-task-synthesis-kim-2026.md
arxiv_id: "2609.07925"
authors:
  - Minseon Kim
  - Zhengyan Shi
  - Emiliano Penaloza
  - Christopher Cui
  - Roger Creus Castanyer
  - Maryam Hashemzadeh
  - Isadora White
  - Jonathan Light
  - Jeonghye Kim
  - Matheus Pereira
  - Darya Moldavskaya
  - Chinmay Singh
  - Fabio Vera
  - Baolin Peng
  - Xingdi Yuan
  - Marc-Alexandre Côté
  - Alessandro Sordoni
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# FrogNano: Training a 4B Coding Agent via Online Task Synthesis

**Kim, Shi, Penaloza, Cui, Creus Castanyer, … Yuan, Côté, Sordoni** (Froggy Team, [[microsoft-research]] Montréal) — [arXiv:2609.07925](https://arxiv.org/abs/2609.07925), cs.AI, v2 9 Sep 2026. Technical report; no venue stated. Project page: microsoft.github.io/debug-gym.

**Summary.** FrogNano is [[qwen]] Qwen3.5-4B post-trained **only with RL**: no SFT, and no stronger model's trajectories, reasoning or patches to imitate. Two ingredients carry it. **Leaf** is a minimal [[claude-code]]-style harness with five typed tools (read, write, edit, glob, bash) where a reply with no tool call ends the episode. **TaskPilot** is an online task generator. It builds tasks (problem statement, gold patch, hidden fail-to-pass tests) from SWE-rebench repository snapshots and runs the *current* checkpoint on each candidate. Tasks the checkpoint always or never solves are rewritten, for example by removing solution-localizing detail or adding a missing behavioral contract, until the resolve rate lands in the target band. Five rounds of about 300 tasks each, roughly 1,500 in total, alternate with 200-update DPPO climbs. The RL uses binary test reward, a success-gated log-length penalty from iteration 3, and no KL or entropy term.

## Key results (Leaf harness, 150 steps, 131k context, mean of 3 seeds)

| Benchmark | FrogNano | Base Qwen3.5-4B (Leaf) |
|---|---|---|
| SWE-bench Verified | **61.5%** | 37.2% (§2) / 39.4% (Fig. 1, App. E) / 43.0% (§5.1) |
| SWE-bench Pro | 37.6% | — |
| Terminal-Bench 2.0 | 31.1% | — |
| PatchEval-Verified | 23.2% | — |

- **The harness matters before training does.** Moving Qwen3.5-4B from the R2E-Gym/SWE-Agent harness to Leaf lifts SWE-bench Verified from **8.3% to 37.2%**. Under R2E-Gym about 96% of its trajectories hit the turn limit. MiniMax-M2.5 scores 66.5% on both harnesses, so the sensitivity is a small-model effect ([[harness-vs-model-attribution]]).
- **Synthetic tasks versus real tasks (Fig. 1).** Per-iteration SWE-bench Verified is 49.1 → 53.1 → 56.9 → 59.1 → 61.5. Training on about 300 real SWE-rebench tasks filtered with the same 4B learnability test reaches 48.0%; about 1,500 such tasks reach 53.4%. Five calibrated synthetic rounds beat that real-data baseline by about 8 points.
- **What "competitive" is measured against.** The paper places FrogNano level with **year-old 32B–100B+ models** and names GPT-5 mini, Grok 4 and Opus 4.1 as systems it "competes with". It reports "competitive with models 6–8× its size" on SWE-bench Pro at a quarter of their cost. It costs **$0.21 per SWE-bench Verified task**, which App. B puts at one-tenth of GPT-5 mini's cost. Current frontier models in Fig. 2 (Claude Opus 4.8/5, GPT-5.5) sit far above it. Comparator scores come from technical reports and leaderboards, not from a matched harness.
- **Transfer and capability.** Evaluated in the unseen bash-only mini-SWE-agent harness, the score rises from 43.8% to 56.4%. The pass@k gap to the base model stays constant as k grows, which the authors read as RL widening what the model can solve, not just sharpening pass@1. A round-robin [[verifier]] over 3 candidates gives 62.8%.
- **Reward hacking.** Regex screening flagged 21.3% of trajectories, and an LLM-judge vote confirmed 2.5% as hacking attempts, mostly weakening the graded tests. The harness and infrastructure blocked all of them, leaving **0 hack-related solves** ([[reward-hacking]]).

> [!note] "No distillation" is scoped to trajectories, not to stronger models
> The authors say plainly that stronger models may *author tasks*; they only supply no behavior for the policy to imitate. The task-generation model is never named, and iteration 5 moved to "a stronger task-generation model" after an initial fifth round gained less. A larger LLM therefore still shapes the curriculum, as task writer rather than teacher. The separate consolidation experiment (62.3%) uses SFT and GPT-5.6-Sol privileged information and is **not** part of FrogNano.

**What it does not show.** The "learnability frontier is important" claim has no ablation against *uncalibrated* synthetic tasks. The only comparison is with real tasks filtered for learnability, so Leaf, online calibration and the generator model's strength are confounded. SWE-bench Verified is also the **validation set** used to decide when each iteration has saturated, so the headline 61.5% is not held out; only the other three benchmarks are. The base-model score is reported three ways (37.2 / 39.4 / 43.0), the iteration-3 score two ways (56.9 in the text, 56.7 in Fig. 1), and the final pass@1 is once given as 60.4%. Training covers Python repositories only, and Leaf is the only training harness.

## Related

- [[agentic-rl]] · [[grpo]] · [[reinforcement-learning]] — DPPO is a GRPO-family trust-region variant; the zero-variance group filter comes from DAPO.
- [[agentic-harness-engineering]] · [[harness-vs-model-attribution]] · [[code-as-harness]] — the 8.3 → 37.2 harness jump is a clean small-model data point.
- [[knowledge-distillation]] · [[on-policy-distillation]] — the path FrogNano avoids; compare [[co-evolving-harnesses-and-models-on-policy-correction-yu-2026]] for the case against imitating an expert's trajectories.
- [[lego-rl-harness-native-reinforcement-learning-coding-agents-du-2026]] — RL through real harnesses, where FrogNano uses a single bespoke one.
- [[agenticqwen-small-agentic-lms-with-dual-data-flywheels-lyu-2026]] · [[agent-world-real-world-environment-synthesis-dong-2026]] — other small-agent and environment-synthesis recipes.
- [[swe-bench]] · [[terminal-bench]] · [[swe-agent]] · [[swe-gym]] · [[claude-code]] · [[minimax]] · [[verifier]] · [[test-time-compute]] · [[llm-as-a-judge]].

## References

- [arXiv:2609.07925](https://arxiv.org/abs/2609.07925) (v2, 9 Sep 2026).
- DPPO: Qi et al., [arXiv:2602.04879](https://arxiv.org/abs/2602.04879). DAPO (Yu et al. 2025); PipelineRL (Piché et al., [arXiv:2509.19128](https://arxiv.org/abs/2509.19128)).
- Task sources and comparators: SWE-rebench (Badertdinov et al.); R2E-Gym ([arXiv:2504.07164](https://arxiv.org/abs/2504.07164)); SWE-smith; BugPilot ([arXiv:2510.19898](https://arxiv.org/abs/2510.19898)); CalibForge ([arXiv:2608.06352](https://arxiv.org/abs/2608.06352)).
- Benchmarks: SWE-bench Verified; SWE-bench Pro (Deng et al., ICML 2026); Terminal-Bench (Merrill et al., ICLR 2026); PatchEval-Verified.
