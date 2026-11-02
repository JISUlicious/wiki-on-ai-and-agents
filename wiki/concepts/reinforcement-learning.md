---
title: Reinforcement Learning
type: concept
created: 2026-05-21
updated: 2026-11-02
sources:
  - deepseek-r1-2025.md
  - rl-an-introduction-sutton-barto-2020.md
  - deepseekmath-shao-2024.pdf
status: draft
importance: high
---

# Reinforcement Learning

**Reinforcement learning (RL)** is the umbrella family of methods that optimize a policy by interacting with an environment and maximizing cumulative reward. In the LLM era, RL has resurfaced as the dominant tool for post-training: aligning models to human preferences ([[rlhf]], [[rlaif]]), eliciting reasoning ([[deepseek-r1]]), and training agents against executable rewards ([[agentic-rl]], [[search-r1]]).

## Why baselines and critics matter

[[rl-an-introduction-sutton-barto-2020|Sutton & Barto]] describe policy gradient methods as learning a parameterized policy directly. REINFORCE is the Monte Carlo form, and subtracting a baseline reduces variance without adding bias. When a learned value function also assesses the action through a one-step return, it is a critic, which adds bias but greatly reduces variance ([[actor-critic]]).

LLM post-training mostly varies this choice of baseline. [[ppo]] uses a learned critic, while [[grpo]] replaces it with the mean reward of a sampled group. Treating GRPO as a return to sample-based baselines is this wiki's framing. See [[policy-optimization-methods]].

## Variants in this wiki

- Classic algorithms: policy gradient, [[ppo]], [[grpo]]
- Preference-based variants: [[rlhf]], [[rlaif]], [[dpo]] (offline analog)
- Reasoning-elicitation RL: [[deepseek-r1]] uses pure RL with verifiable rewards
- Agentic RL: [[agentic-rl]], [[search-r1]] reward task success in multi-step tool-using rollouts

Related: [[rlhf]], [[ppo]], [[grpo]], [[agentic-rl]], [[deepseek-r1]], [[search-r1]]

## References
- [[rl-an-introduction-sutton-barto-2020]]
- [[deepseekmath-shao-2024]]
- [[deepseek-r1-2025]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]] for full grouping.

- [[embarrassingly-simple-self-distillation-zhang-2026]] — Embarrassingly Simple Self Distillation Improves [[code-generation|Code Generation]]
- [[pivotrl-high-accuracy-agentic-post-training-yi-2026]] — PivotRL High Accuracy Agentic Post Training at Low
- [[primer-in-post-training-reasoning-data-li-2026]] — A Primer in Post Training Reasoning Data What
- [[self-distilled-rlvr-yang-2026]] — Self Distilled RLVR
- [[vector-policy-optimization-bahlous-boldi-2026]] — Vector Policy Optimization Training for Diversity Improves Test
- [[policy-optimization-methods]] — Policy Optimization Methods in LLM Post Training
- [[back-on-track-aligning-rewards-and-states-shao-2026]] — PAPO: stable RL for reasoning in diffusion LLMs
- [[human-like-autonomy-self-play-pinch-human-data-cornelisse-2026]] — self-play driving + a pinch of human data as regularizer
- [[beyond-scalar-rewards-z-reward-jin-2026]] — Z-Reward: [[reward-modeling|reward modeling]] via score distributions

> [!note] Learning without backprop
> RL policies can be trained with [[backprop-free-learning|backpropagation-free rules]]; see [[diffusing-blame-yamada-2026]] for ED-PPO vs BP-PPO vs DFA-PPO across Brax locomotion and Craftax.

## Lineage sources

- [[actor-critic]] — actor–critic methods (Barto, Sutton & Anderson 1983)
- [[rl-an-introduction-sutton-barto-2020]] — Sutton & Barto, chapter 13
