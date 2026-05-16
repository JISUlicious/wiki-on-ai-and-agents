---
title: GRPO (Group Relative Policy Optimization)
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - memory-r1-yan-2025.md
status: draft
importance: medium
tags: []
---

Group Relative Policy Optimization (GRPO) is a reinforcement learning algorithm introduced by DeepSeek for training language models, closely related to but architecturally distinct from [[ppo|PPO]]. Where PPO requires a learned value function (critic) to estimate per-token advantages, GRPO eliminates the critic entirely and instead uses *group-relative* normalization: for each prompt the policy samples a group of `G` candidate responses, each is scored by the reward model, and the advantage of each response is computed as its reward minus the group mean (often divided by the group standard deviation). This removes the memory cost of a separate value network — significant for LLM-scale models — and tends to produce more stable updates because the baseline is empirically derived from on-policy samples rather than a moving learned estimate. GRPO has rapidly become a default for LLM RL fine-tuning on tasks with verifiable or model-scored rewards.

## Where it appears

- [[memory-r1]] — Yan et al. (2025) train an agent's memory-write/edit/delete policy with GRPO, treating each memory operation as an action whose reward is the downstream answer correctness.
- [[ppo]] — the immediate predecessor and conceptual contrast point; GRPO swaps PPO's critic for a group-relative baseline.
- [[reinforcement-learning]] — sits in the policy-gradient family alongside [[ppo]] and [[dpo]].
- [[rl-memory-policy]] — RL formulations where the action space consists of memory operations rather than tokens.

## References

- `memory-r1-yan-2025.md`
