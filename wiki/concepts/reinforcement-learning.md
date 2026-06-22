---
title: Reinforcement Learning
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - deepseek-r1-2025.md
status: draft
importance: high
tags:
  - rl
---

**Reinforcement learning (RL)** is the umbrella family of methods that optimize a policy by interacting with an environment and maximizing cumulative reward. In the LLM era, RL has resurfaced as the dominant tool for post-training: aligning models to human preferences ([[rlhf]], [[rlaif]]), eliciting reasoning ([[deepseek-r1]]), and training agents against executable rewards ([[agentic-rl]], [[search-r1]]).

- Classic algorithms: policy gradient, [[ppo]], [[grpo]]
- Preference-based variants: [[rlhf]], [[rlaif]], [[dpo]] (offline analog)
- Reasoning-elicitation RL: [[deepseek-r1]] uses pure RL with verifiable rewards
- Agentic RL: [[agentic-rl]], [[search-r1]] reward task success in multi-step tool-using rollouts

Related: [[rlhf]], [[ppo]], [[grpo]], [[agentic-rl]], [[deepseek-r1]], [[search-r1]]

## References
- [[deepseek-r1-2025]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]] for full grouping.

- [[embarrassingly-simple-self-distillation-zhang-2026]] — Embarrassingly Simple Self Distillation Improves Code Generation
- [[pivotrl-high-accuracy-agentic-post-training-yi-2026]] — PivotRL High Accuracy Agentic Post Training at Low
- [[primer-in-post-training-reasoning-data-li-2026]] — A Primer in Post Training Reasoning Data What
- [[self-distilled-rlvr-yang-2026]] — Self Distilled RLVR
- [[vector-policy-optimization-bahlous-boldi-2026]] — Vector Policy Optimization Training for Diversity Improves Test
- [[policy-optimization-methods]] — Policy Optimization Methods in LLM Post Training
- [[back-on-track-aligning-rewards-and-states-shao-2026]] — PAPO: stable RL for reasoning in diffusion LLMs
