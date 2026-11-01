---
title: "Sharpening Tax in Post-Training"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - sharpening-tax-in-post-training-oh-2026.md
arxiv_id: "2610.01509"
authors:
  - Changdae Oh
  - Qi Zeng
  - Qi Qi
  - Andrey Zhmoginov
  - Deren Lei
  - Yun He
  - Hoang Phan
  - Hangoo Kang
  - Azalia Mirhoseini
  - Sharon Li
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Sharpening Tax in Post-Training

**Changdae Oh, Qi Zeng, Qi Qi, Andrey Zhmoginov, Deren Lei, Yun He, Hoang Phan, Hangoo Kang, Azalia Mirhoseini, Sharon Li** (Meta Superintelligence Labs, [[meta]]; UW–Madison; NYU; [[stanford-university|Stanford]]): [arXiv:2610.01509](https://arxiv.org/abs/2610.01509), v1, cs.AI, 1 Oct 2026. Preprint; no venue.

## Summary

Does RL post-training create new capabilities, or does it only **sharpen** the base model's distribution, raising pass@1 at the cost of coverage (pass@K)? Earlier evidence, such as Yue et al. 2025, comes mostly from math and code. There, pretraining exposure is huge and checking only the final answer lets lucky guesses count. The authors argue agentic tasks are the harder test. Structured multi-turn tool calls are rare in pretraining, so post-training might be *needed* just to make success reachable. They test this on three benchmarks that check intermediate state: BFCL v4 multi-turn base, [[webshop|WebShop]] and ACEBench.

The surprising result: **base models with a light inference harness are capable agents**. Their pass@1 is much lower, but as K grows to 128 they often **overtake their post-trained counterparts in coverage**. For example, on WebShop gemma-4-31B base exceeds 85% pass@128, versus 56% for the -it model. Mechanistically, post-training **bimodalizes** per-task success: tasks become always-solved or never-solved, and the middle band of "solvable given compute" shrinks (87.6% → 30.0% on WebShop for gemma-4-31B). That buys consistency and sample efficiency but gives up the payoff from parallel test-time scaling.

## Key points

- **Scope.** 14 base/post-trained pairs from 4 families (Gemma-4, Ministral-3, Qwen2.5, Qwen3.5; 3B–35B effective parameters) × 3 benchmarks = 42 cases.
- **The Sharpening Tax.** A scalar that measures the loss in test-time scalability, in two variants: raw (TaxA) and accuracy-headroom-calibrated (TaxS). **TaxS(128) > 0 in 36 of 42 combinations.** The tax is largest for the largest checkpoints (gemma-4-31B). For the smallest models it is often *negative* at k ≤ 8, so sharpening helps when compute is tight.
- **Larger models cross over sooner.** On WebShop the crossover budget k* falls from more than 128 (Gemma-4 4B) to about 3 (31B).
- Among the families, Qwen3.5 sharpens the least and Gemma-4 the most. The tax can be estimated from a few rollouts and correlates with other metrics.
- **The harness matters for base models.** The family-averaged base pass@32 goes from 15.19 to 49.13 on BFCL with the harness, and from 39.48 to 49.15 on WebShop.
- **PTGS (posterior-tempered group sampling).** A Bayesian sampler sets a per-prompt temperature from estimated difficulty: it cools mastered prompts and heats hard ones, so more groups have mixed outcomes and give GRPO a nonzero advantage. Tested with Qwen2.5-7B-Instruct on **Sokoban and FrozenLake** (RAGEN setup; PPO and GRPO; 200 steps; 5 seeds). Task-specific RL also pays the tax, and PTGS improves pass@1 *and* pass@128 while lowering TaxS.

## What it does not show

> [!warning] Newsletter vs paper
> The newsletter lists PTGS as "the fix" right after BFCL / ACEBench / WebShop. PTGS is evaluated **only on Sokoban and FrozenLake**, not on the three benchmarks where the tax was measured.

- The study is **observational**: training data and recipes for the post-trained checkpoints are undisclosed, so "post-training" lumps together SFT, DPO and RL.
- **Thinking mode is turned off** for all post-trained models (with one ablation, Fig. 10). That probably understates what the post-trained models can do on the long-horizon tasks.
- The metric needs a binary success signal, so open-ended tasks are out of scope.

## Concepts & entities

- [[reinforcement-learning]] · [[agentic-rl]] · [[grpo]] · [[ppo]]: the sharpening-vs-expansion debate moved to agents.
- [[understanding-reasoning-pretraining-posttraining-shen-2026]]: **consistent**. In that page's chess setting, pass@16 stays flat or degrades under RL while pass@1 rises, and "wrong-mode amplification" offers a mechanism. This paper extends the pattern to agentic tool use. See also [[recirculation-mozer-2026]] (pass@1 vs pass@128 read via Yue et al.) and [[vector-policy-optimization-bahlous-boldi-2026]] (methods that target pass@k).
- [[test-time-compute]] · [[inference-time-compute]] · [[self-consistency]]: coverage at large K is the quantity being taxed.
- [[quality-diversity]] · [[structured-output-collapses-diversity-parikh-2026]]: diversity loss after alignment.
- [[agent-evaluation]]: pass@k vs pass^k. Bimodalization raises pass^k and lowers pass@k.
- [[webshop]] · [[function-calling]].
- Same lab and batch: [[mimesis-user-simulators-training-environments-phan-2026]]. Same benchmark: [[critical-state-rl-trainable-states-tool-use-chen-2026]].

## References

- [arXiv:2610.01509](https://arxiv.org/abs/2610.01509) · Code: changdaeoh/sharpening-tax
- Yue et al. 2025, "Does RL really incentivize reasoning capacity beyond the base model?" (not ingested)
