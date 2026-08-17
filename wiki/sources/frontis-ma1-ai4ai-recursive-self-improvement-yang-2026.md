---
title: "Frontis-MA1: Training an AI4AI Model towards Recursive Self-Improvement in Machine Learning Engineering"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2607.28568"
authors:
  - Junlin Yang
  - Che Jiang
  - Yu Fu
  - Tianwei Luo
  - Can Ren
  - Weizhi Wang
  - Kaikai Zhao
  - Ning Ding
  - Bowen Zhou
  - Kaiyan Zhang
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Frontis-MA1: Training an AI4AI Model towards Recursive Self-Improvement in Machine Learning Engineering

**Junlin Yang, Che Jiang, Yu Fu, Tianwei Luo, Can Ren, Weizhi Wang, Kaikai Zhao, Hongyi Liu, Yuxin Zuo, Yuru Wang, Yuchen Fan, Kai Tian, Zhenzhao Yuan, Xiaojian Lin, Li Sheng, Rushi Qiang, Guoli Jia, Xingtai Lv, Ermo Hua, Dianqiao Lei, Youbang Sun, Ning Ding, Bowen Zhou, Kaiyan Zhang** — Horizon Research / Frontis.AI, [[tsinghua-university]], [[zhejiang-university]], [[shanghai-jiao-tong-university]], Georgia Institute of Technology. [arXiv:2607.28568](https://arxiv.org/abs/2607.28568), cs.CL, 30 Jul 2026, CC BY 4.0. Preprint; no venue given.

## Summary

The framing is **AI4AI** — using AI to build AI — with **recursive self-improvement (RSI)** as the stated endpoint and **machine learning engineering (MLE)** as the executable testbed. The contribution is not a new algorithm but a **full stack**, released open, with one design commitment running through all three layers: the *same four atomic program-evolution operators* — `Draft`, `Improve`, `Debug`, `Crossover` — are the SFT targets, the RL targets, and the moves the inference-time search composes. That shared interface is what the authors call **meta-evolution**: verified evolutionary trajectories post-train the very model that acts as the variation engine of the search, so "the improver itself is trained."

The three layers are **OpenMLE-Gym** (5,758 quality-gated executable tasks with sandboxed execution, structured six-mode feedback, and task-specific evaluators), **OpenMLE-ERL** (execution-grounded SFT then RL on those operators), and **OpenMLE-Evo** (long-horizon evolutionary search using structured experience records, three-factor parent selection, and operator-conditioned memory). The trained model is **Frontis-MA1-35B** (MA1 = Meta-evolution Agent, generation 1), post-trained from Qwen3.6-35B-A3B, with **Frontis-MA1-30B** (from Qwen3-30B-A3B-Thinking-2507) as a controlled cross-backbone replication.

Two claims are cleanly separated by the experiment design, and this separation is the paper's real methodological strength: **model gains** (swap the model, hold the harness) and **harness gains** (swap the harness, hold the model). Both are positive, and they compose.

## Key points

**Headline results, MLE-Bench Lite** (official 22-task split, three independent runs, **12 h per task on a single RTX 4090 capped at 12 GB VRAM** — the authors note this is a smaller sandbox budget than most reported MLE-Bench evaluations). *Valid Rate* = mean tasks yielding a valid submission out of 22; *Medal Average* = mean fraction of tasks earning any Kaggle medal; *Human Rank* = fraction of human leaderboard participants surpassed.

| Model | Framework | Valid Rate | Medal Average | Human Rank |
|---|---|---|---|---|
| Qwen3.6-35B-A3B (base) | OpenMLE-Evo | 19.67/22 | 39.39% | 0.5828 |
| **Frontis-MA1-35B** | OpenMLE-Evo | 21.67/22 | **60.61%** | 0.7647 |
| **Frontis-MA1-35B** | OpenMLE-Evo-Max | 22.00/22 | **71.21%** | **0.8126** |
| Qwen3-30B-A3B-Thinking (base) | OpenMLE-Evo | 17.33/22 | 34.85% | 0.5573 |
| Frontis-MA1-30B | OpenMLE-Evo | 21.67/22 | 53.03% | 0.7055 |
| Frontis-MA1-30B | OpenMLE-Evo-Max | 22.00/22 | 66.67% | 0.8053 |
| *GPT-5.5* | *Codex* | 21.00/22 | 68.18% | 0.7833 |
| *GPT-5.6 Sol* | *Codex* | 22.00/22 | 72.73% | 0.8891 |
| *Kimi K3* | *Claude Code* | 22.00/22 | 72.73% | 0.8574 |
| *[[claude-opus-4-8]]* | *[[claude-code]]* | 22.00/22 | 63.64% | 0.8219 |

Post-training alone, harness held fixed, is worth **+21.22 pp** Medal Average on the 35B (and +18.18 pp on the 30B) — that is the clean model-level number. **OpenMLE-Evo-Max** adds MLE-Bench-disjoint cross-task priors distilled from public competition artifacts plus asynchronous multi-GPU parallel search at unchanged total sandbox compute, taking the 35B system to 71.21%.

> [!warning] The headline margin is inside the noise
> Appendix D.1 reports std over the three evaluation epochs: Frontis-MA1-35B + Evo-Max is **71.21% ± 8.57 pp**, and the base is 39.39% ± 5.67 pp. The paper's "exceeds GPT-5.5 + Codex by 3.03 percentage points" is **well under one standard deviation** — 3.03 pp on 22 tasks × 3 runs is two task-outcomes out of 66. The large post-training gaps (+21.22, +18.18 pp) survive this; the frontier-model comparisons do not.

**Harness-level, model held fixed.** OpenMLE-Evo beats general-purpose coding-agent scaffolds across four model families, and beats the original AIRA-Evo on the same Frontis-MA1-35B checkpoint (53.03% → 60.61%):

| Model | Claude Code / Codex | OpenMLE-Evo | OpenMLE-Evo-Max |
|---|---|---|---|
| GLM-5.2 | 59.09% | 62.12% | 66.67% |
| MiniMax M3 | 54.55% | 59.09% | 65.15% |
| Kimi K2.6 | 59.09% | 66.67% | — |
| MiniMax M2.7 | 45.50% | 50.00% | — |

**Why the harness wins — measured, not asserted.** Matched single-worker trajectories from the same checkpoint, same seed, same 12-h budget, 66 task–runs per harness, vs. original AIRA-Evo: total model tokens **129.3M → 75.3M (−41.7%)** and prompt tokens **83.5M → 41.5M (−50.3%)**, while evaluated nodes fall only 3430 → 3004 (−12.4%). The saving is per-expansion, not from stopping early. Yield rises at the same time: new-best validation updates 229 → **246**, i.e. **1.77 → 3.27 per million tokens (+84.3%)**, and the fraction of `Improve` calls that set a new best goes **4.73% (44/931) → 9.36% (72/769)**. The mechanism is bounded, operator-conditioned context: mean serialized `Improve` prompt **102.8K → 35.7K** characters (−65.3%), 99th percentile **389.0K → 54.3K** (−86.1%); `Crossover` mean 140.4K → 55.3K, p99 419.2K → 78.4K. **Compression is strongest in the tail** — long histories are no longer re-serialized into every request.

**The three memory/search design choices** that produce that compression:
- **Structured experience cards** stored deterministically per node (validation score, improvement over parent, method family, runtime, rank, incumbent status, direction novelty), plus a global experience board of family-level success/failure stats, underexplored directions, and recurring error signatures.
- **Operation-triggered memory synthesis** — AIRA-Evo eagerly LLM-summarizes *every* evaluated node; OpenMLE-Evo defers natural-language synthesis until an `Improve`/`Crossover`/`Debug` call has actually selected its nodes, then synthesizes only for those, and caches. Summaries are produced *after* the decision context that should shape them is known.
- **Three-factor parent selection** over score / parent-relative gain / method-family novelty (weights 1.0 / 0.6 / 0.3) instead of score-only. Worked example: a parent ranking only 6th by score but 1st by gain has its selection probability raised **10.47% → 17.09%** in the same ten-parent pool (+63.2% relative); its `Improve` child reached held-out AUC 0.99386. The point is not forcing weaker branches to win but keeping structurally distinct, high-gain branches **actionable long enough to be selected**.

**Training details worth keeping.** SFT corpus is **26,259 examples** — 17,245 full-response `Draft` examples from a parallel sampling path plus 9,014 trajectory-step examples from an evolutionary path — collected under a budget-adaptive stopping rule (accepted-example quota *or* execution budget, so sparse-success tasks get more attempts). RL replaces the usual [[grpo]]-style group-normalized advantage with (a) **adaptive score bounds** derived from each task's on-policy score frontier, so meaningfully different programs do not collapse to near-identical rewards under fixed leaderboard extrema, and (b) an **entropic advantage** that amplifies reward gaps near the top of each rollout group — MLE is judged by the *best* program found, so a barely-viable submission should not earn the same signal as a strong one. Rollouts are asynchronous (groups admitted on completion rather than waiting for the slowest sandbox job).

**Reward hacking was observed and required an explicit guard.** On difficult tasks, reward plateaued low because models learned to take the sample submission, **randomly shuffle it, and submit that**. The fix: an o3-mini [[llm-as-a-judge|LLM judge]] runs a reward-hack check *before* sandbox execution; detected hacks bypass execution and receive reward **−0.5**. Worth noting that the guard is a separate frozen model, not the trained policy.

**Long-horizon behavior.** Gains come from continued structural improvement, not just finding a first executable solution. On `leaf-classification`, late `Improve`+`Crossover` operations produce **85.0%** of total validation gain (held-out Human Rank 0.9455, Bronze; strongest comparison model earned no medal); on `mlsp-2013-birds`, **91.9%** (held-out 0.8889, Silver). Final-test Medal Rate 71.21% actually **exceeds** validation Medal Rate 68.18% — the selected solutions generalize. The medal-tier decomposition shifts toward Gold rather than merely crossing the Bronze threshold.

**Cross-modality and transfer.** Across five modality groups on MLE-Bench Lite, Frontis-MA1-35B raises mean Human Rank in **all five** and never lowers group-level Medal Rate; the 14 additional medals spread across every group (image/text/tabular/audio/multimodal = +2/+4/+1/+4/+3). On **NatureBench Lite** (10-task subset of the 90-task NatureBench, tasks distilled from Nature-family papers, hidden host-side evaluators, web search disabled, 4-h budget per task), both components transfer independently:

| Model | Harness | Surpass-SOTA (g>0.1) | Match-SOTA (g≥0) |
|---|---|---|---|
| Qwen3.6-35B-A3B | original AIRA-Evo | 10% (1/10) | 20% (2/10) |
| Qwen3.6-35B-A3B | OpenMLE-Evo NB adapter | 20% (2/10) | 50% (5/10) |
| **Frontis-MA1-35B** | OpenMLE-Evo NB adapter | 30% (3/10) | 70% (7/10) |

Hold the adapter, swap the model: +20 pp Match-SOTA. Hold the model, swap the harness: +30 pp Match-SOTA. The combined system matches GPT-5.4, GLM-5.1 and MiniMax-M3 on this subset — but note Claude Opus 4.7 and GLM-5.2 with Claude Code both hit **100% Match-SOTA / 70% Surpass-SOTA**, so this is transfer evidence, not a frontier result.

**Stated limitations** — five capability boundaries, and the authors are explicit that OpenMLE **does not realize RSI**: (1) the learning signal is the measured outcome of an executed solution, so the system "is better equipped to optimize solutions than to judge which ideas deserve to be pursued"; (2) trained operators are composed by an *external* harness, bounding the actions the model can initiate; (3) the agent improves external ML artifacts, not language models themselves; (4) **evolution operates over candidate solutions while the evolutionary system itself stays fixed** — making the search an object of evolution is future work; (5) parent selection uses three hand-chosen factors with fixed weights, leaving most of the recorded experience unused. The related-work section closes with the same disclaimer: a testbed for studying progress toward RSI, "rather than a claim that OpenMLE realizes general, autonomous recursive self-improvement."

## Related

- [[self-improving-agent]] — the target pattern; this trains the *improver*, but explicitly stops short of the system evolving itself.
- [[godel-machine]] · [[darwin-godel-machine]] · [[adas]] — the RSI lineage the paper positions against; DGM evolves the scaffold with a frozen FM, this evolves the FM with a fixed scaffold (the mirror image).
- [[agentic-rl]] — the training paradigm: execution-grounded RL over long-horizon agent tasks.
- [[grpo]] — the baseline advantage estimator, replaced here by adaptive bounds + entropic upper-tail weighting.
- [[reinforcement-learning]] · [[outcome-based-reward]] — reward is the sandbox-verified task score; no learned reward model.
- [[reward-hacking]] — directly observed (shuffled sample submissions) and guarded with a pre-execution judge at −0.5 reward.
- [[llm-as-a-judge]] — used twice: task quality gating for the gym, and the reward-hack check during RL.
- [[verifier]] — the task-specific `metric.py` evaluator is the ground-truth verifier the whole stack is built on.
- [[fine-tuning]] · [[knowledge-distillation]] — execution-grounded SFT distills teacher solutions to broaden sampled support before RL.
- [[inference-time-compute]] · [[test-time-compute]] — the long-horizon search converts a 12-h budget into sustained gains rather than redundant sampling.
- [[memory-management]] · [[experiential-learning]] — structured experience cards + operation-triggered synthesis; the token savings come from bounding context, not from remembering less.
- [[mixture-of-experts]] — both backbones are MoE (`A3B` = 3B active parameters).
- [[code-generation]] · [[llm-agent]] · [[agentic-harness-engineering]] — the operators are code transformations; the harness-vs-model separation is the paper's cleanest experimental idea.
- [[claude-code]] · [[claude-opus-4-8]] — general-purpose harness/model baselines in the comparison tables.
- [[tsinghua-university]] · [[zhejiang-university]] · [[shanghai-jiao-tong-university]] — affiliations.
- [[mle-bench]] — the primary benchmark (no page yet).

## References

- [arXiv:2607.28568](https://arxiv.org/abs/2607.28568)
