---
title: PPO (Proximal Policy Optimization)
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - memory-r1-yan-2025.md
status: draft
importance: medium
tags: []
---

Proximal Policy Optimization (PPO) is a policy-gradient reinforcement learning algorithm introduced by Schulman et al. at OpenAI in 2017, and for years the de facto workhorse of LLM RL post-training — including [[rlhf|RLHF]] and tool-use training. PPO improves on vanilla policy gradient by *clipping* the ratio between the new and old policy probabilities for each action: updates that move the policy too far from the sampling distribution are truncated, preventing the destructive large-step updates that destabilize unconstrained policy gradients. The algorithm pairs an actor (the LM being trained) with a critic value network used to estimate advantages. PPO was the RL algorithm behind [[instructgpt|InstructGPT]] and the original ChatGPT alignment pipeline, and remains widely used; however, it is increasingly being supplemented or replaced by [[dpo|DPO]] (which sidesteps RL entirely via a closed-form preference objective) and [[grpo|GRPO]] (which keeps the on-policy RL formulation but removes the critic in favor of a group-relative baseline).

## Where it appears

- [[rlhf]] — PPO is the canonical RL stage in the standard SFT → RM → PPO pipeline.
- [[instructgpt]] — first prominent application of PPO to align an LLM with human preferences.
- [[memory-r1]] — Yan et al. (2025) compare PPO and [[grpo]] for training memory-operation policies.
- [[grpo]] and [[dpo]] — the two main contemporary alternatives that have eroded PPO's dominance.
- [[reinforcement-learning]] — sits in the policy-gradient family.

## References

- `memory-r1-yan-2025.md`
