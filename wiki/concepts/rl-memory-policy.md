---
title: RL Memory Policy
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - memory-r1-yan-2025.md
status: draft
importance: medium
tags:
  - 2025
---

An RL memory policy treats memory lifecycle operations — `ADD`, `UPDATE`, `DELETE`, `NOOP` — as actions chosen by a learned policy, rather than as hard-coded heuristics (e.g., "always append" or "evict oldest"). The agent is trained with reinforcement learning so that *when to remember* and *when to forget* emerge from downstream task rewards rather than from manual rules. This reframes memory management as a control problem over a structured external store. Pioneered by [[memory-r1]] (Yan 2025), which trains the memory controller with PPO and GRPO and shows that learned policies outperform retrieval-only and append-only baselines on long-horizon tasks.

## Where it appears

- [[memory-r1]] — Direct introduction; learns ADD/UPDATE/DELETE/NOOP via RL.
- [[reinforcement-learning]] — The training paradigm.
- [[grpo]] — One of the optimizers used.
- [[ppo]] — The other optimizer used.
- [[memory-management]] — RL policy is one strategy in the broader memory-control design space.

## References

- [[memory-r1-yan-2025]] — Memory-R1: Reinforcement Learning for Memory Management in LLM Agents (Yan 2025).
