---
title: "From System 1 to System 2: A Survey of Reasoning Large Language Models"
type: source
created: 2026-10-10
updated: 2026-10-10
sources:
  - from-system-1-to-system-2-survey-li-2025.md
arxiv_id: "2502.17419"
venue: arXiv survey
authors:
  - Zhong-Zhi Li
  - Duzhen Zhang
  - Ming-Liang Zhang
  - Jiaxin Zhang
first_author: Zhong-Zhi Li
year: 2025
tags:
  - 2025
  - paper
  - survey
status: complete
importance: medium
---

# From System 1 to System 2: A Survey of Reasoning LLMs

**Source**: `sources/from-system-1-to-system-2-survey-li-2025.md` ([arXiv:2502.17419](https://arxiv.org/abs/2502.17419); v1 2025-02-24; v6 read)
**Authors**: Zhong-Zhi Li, Duzhen Zhang, Ming-Liang Zhang, Jiaxin Zhang, et al.

## Summary

The survey frames the field's arc in dual-process terms:
- "**Foundational LLMs excel at fast decision-making** but lack the depth for complex reasoning."
- **Reasoning LLMs** such as [[o1|OpenAI o1/o3]] and [[deepseek-r1|DeepSeek-R1]] "closely mimic the deliberate reasoning of System 2".

It traces early **System 2 technologies** combined with foundational LLMs:
- symbolic logic;
- MCTS;
- reinforcement learning;
- [[chain-of-thought-prompting|CoT]];
- [[tree-of-thoughts|ToT]] and search;
- [[process-reward-model|process reward models]] and verifiers.

It then covers how reasoning LLMs are built (data, RL, test-time search), benchmarks, and efficient reasoning.

## Why it matters

- **It is the mainstream LLM use of the metaphor.** System 1 is *default next-token generation*. System 2 is *explicit, extended reasoning at inference time*, which is what [[large-reasoning-model|reasoning models]], [[test-time-compute]] and [[reasoning-effort|effort settings]] provide.
- **Under that framing, the word "System 1" means an ordinary chat LLM.** A [[system-one-model|"System One model"]] narrows it further: one pass, bounded answers, no generated text at all. See [[system-one-models-and-dual-process-thinking]].

## Concepts & entities

[[dual-process-theory]] · [[large-reasoning-model]] · [[test-time-compute]] · [[chain-of-thought-prompting]] · [[tree-of-thoughts]] · [[process-reward-model]] · [[o1]] · [[deepseek-r1]]
