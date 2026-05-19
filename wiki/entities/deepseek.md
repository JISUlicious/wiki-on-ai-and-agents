---
title: DeepSeek
type: entity
created: 2026-05-17
updated: 2026-05-17
sources:
  - deepseek-r1-2025.md
status: draft
importance: medium
tags:
  - org
---

DeepSeek is a Chinese AI lab known for releasing open-weights frontier-class language models. Their lineage includes DeepSeek-V2/V3 base models and the [[deepseek-r1]] reasoning model. The lab also originated [[grpo]] (Group Relative Policy Optimization) via the DeepSeekMath paper, a now-widely-used RL algorithm for LLM post-training.

DeepSeek's open releases — particularly the R1 weights and recipe — substantially shifted the global research landscape on [[agentic-rl]] and verifier-driven reasoning, demonstrating that strong long-CoT behavior can be reached without proprietary RLHF stacks.

## Notable work

- [[deepseek-r1]] — reasoning-focused open-weights model trained with [[grpo]] and rule-based verifiable rewards (see [[deepseek-r1-2025]]).
- DeepSeekMath — original venue introducing [[grpo]].
- DeepSeek-V3 — large MoE base model underlying R1.
- Contributions to [[agentic-rl]] practice via published recipes.

## References

- [[deepseek-r1-2025]]
