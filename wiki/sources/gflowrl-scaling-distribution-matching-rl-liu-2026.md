---
title: "GFlowRL: Scaling Distribution-Matching RL to Large Language Models"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - gflowrl-scaling-distribution-matching-rl-liu-2026.md
arxiv_id: "2607.13394"
year: 2026
authors:
  - Xiaodong Liu
  - Michael Xu
  - Jack W. Stokes
  - Paul Smolensky
  - Jianfeng Gao
first_author: Xiaodong Liu
venue: arXiv (cs.CL) preprint
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# GFlowRL: Scaling Distribution-Matching RL to Large Language Models

**Xiaodong Liu, Michael Xu, Jack W. Stokes, Paul Smolensky, Doug Burger, Jianfeng Gao** (Microsoft Research) — [arXiv:2607.13394](https://arxiv.org/abs/2607.13394) (v1, 15 Jul 2026, cs.CL). Preprint; no venue listed. Code promised at `github.com/microsoft/gflowrl`.

## Summary

GFlowNets match a reward *distribution* instead of maximizing reward, which preserves diverse reasoning paths rather than collapsing onto dominant modes. Prior work (FOR, FlowRL) brought the trajectory-balance objective to LLMs, but it depends on a learned prompt-conditional partition function `Z_φ(x)` — in FlowRL, a 3-layer MLP on top of the prompt representation. This paper's diagnosis is that at post-training scale — where model size, rollout horizon, reward noise, and distributed-systems complexity all grow together — that auxiliary network becomes a source of gradient instability rather than a useful normalizer. A small randomly-initialized MLP has to learn a complex prompt-conditional target while lagging far behind a much larger pretrained policy.

The central finding is a negative result turned into a simplification: the learned partition function, "previously treated as essential," can be replaced by an **in-batch Monte Carlo estimate of log Z computed from the rollout group GRPO already samples per prompt**. No extra network, no extra optimizer state. The evidence that `Z_φ` was never doing real work is a controlled ablation — **FlowRL-RandomLogZ**, which substitutes random draws matching the empirical distribution of `log Z_φ`, actually slightly *beats* standard FlowRL on math (36.19 vs. 35.63 average). On a synthetic three-Gaussian target the two behave nearly identically. The learned normalizer, in other words, contributes little beyond a noisy scalar bias.

GFlowRL keeps the reward-distribution-matching semantics of trajectory balance and adds two stabilizers: **importance-sampling correction** for rollout/trainer drift, and **asymmetric flow-gap clipping** for outlier residuals. The gradient-norm evidence is stark: over 421 training steps, GRPO and GFlowRL hold means of 0.24 and 0.07 with maxima below 6.2, while FlowRL shows a mean of 3.2 × 10^14 including 55 explosion events.

## Key points

**Math, dense (Qwen2.5-7B, Avg@16 across AIME24/25, AMC23, MATH500, Minerva, Olympiad):**

| Method | Average |
|---|---|
| Backbone | 23.02 |
| REINFORCE++ | 31.52 |
| PPO | 31.98 |
| GRPO | 32.48 |
| FlowRL | 35.63 |
| FlowRL-RandomLogZ | 36.19 |
| **GFlowRL** | **40.92** |

GFlowRL exceeds GRPO by **+8.44** average points. MATH500 goes 54.47 → 76.89; AIME24 4.38 → 17.29.

**Code, dense (DeepSeek-R1-Distill-Qwen-7B):** LiveCodeBench Avg@16 30.68 → **38.62**; Codeforces rating 886.68 → **1646.21** (19.4% → 88.0% percentile); HumanEval+ 80.90 → **84.93**. Both GFlowNet-style methods open a 200–330 Elo advantage over GRPO.

**Codeforces at 14B — the headline number is confirmed: 2048 Elo.** Beats DeepCoder-14B by +112, FlowRL-14B by +144, and OpenAI o1 (1891) by +157; within **25 Elo of o3-mini (2073)**.

**MoE — the "first to scale across dense and sparse" claim is confirmed and is the paper's strongest result:**
- Qwen3-30B-A3B reaches **1999 Codeforces Elo with only 3B active parameters** (+108 over o1, within 74 of o3-mini); math average 74.52 → 78.32 (+3.80 over backbone, +2.54 over GRPO).
- Qwen3-235B-A22B, hyperparameters reused unchanged from 30B: math average 82.70 → **85.06**, vs. GRPO's 83.84.
- **FlowRL fails to converge on both MoE backbones**, producing no usable checkpoints — attributed to the partition network's instability under nondeterministic routing.

**Red-teaming (noisy/sparse rewards), ASR@1:** AdvBench **82.5%** avg and HarmBench **79.5%** avg, beating prior SOTA multi-turn attacker SEMA by +2.4 and +4.5 points. FlowRL fails to converge here too and produces no usable attacker. Victim models: Qwen2.5-3B, Llama-3.1-8B, GPT-4.1-mini.

The authors' framing of the payoff: removing the high-variance `log Z_φ` term turns parity on math/code into a strict capability advantage — "GFlowRL covers the union of tasks where either GFlowNet-style or GRPO/PPO training works."

## Concepts & entities

- [[policy-optimization-methods]] — GFlowRL is a distribution-matching alternative to reward-maximizing RL; the in-batch log Z estimate makes it a near-drop-in modification of the GRPO rollout loop.
- [[ppo]] — one of the reward-maximization baselines (alongside GRPO/GRPO+ and REINFORCE++); GFlowRL beats it on every reported benchmark.
- [[mixture-of-experts]] — the sparse-architecture result is the paper's main scaling claim; nondeterministic MoE routing inflates gradient variance and is exactly what breaks FlowRL's auxiliary partition network.
- [[humaneval]] — HumanEval+ is one of the three code benchmarks (80.90 → 84.93).
- [[reward-hacking]] — mode collapse under reward maximization is the failure this line of work targets; distribution matching preserves diverse reasoning paths by construction.
- [[agentic-rl]] — same post-training regime; the red-teaming setting is multi-turn attacker training, the most agentic of the three domains evaluated.

## References

- [arXiv:2607.13394](https://arxiv.org/abs/2607.13394)
- Original source text: `sources/gflowrl-scaling-distribution-matching-rl-liu-2026.md`
