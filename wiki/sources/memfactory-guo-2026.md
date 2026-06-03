---
title: "MemFactory: Unified Inference & Training Framework for Agent Memory"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - memfactory-guo-2026.md
arxiv_id: "2603.29493"
authors:
  - Ziliang Guo
  - Ziheng Li
  - Bo Tang
  - Feiyu Xiong
  - Zhiyu Li
first_author: Ziliang Guo
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# MemFactory: Unified Inference & Training Framework for Agent Memory

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

MemFactory is presented as the first unified, highly modular training and inference framework specifically designed for memory-augmented LLM agents. The motivation: applying reinforcement learning to optimize memory operations (extraction, updating, retrieval) is a promising direction, but existing implementations are fragmented and task-specific, with no shared infrastructure for integration, training, and evaluation. Inspired by unified fine-tuning frameworks like LLaMA-Factory, MemFactory abstracts the memory lifecycle into atomic, plug-and-play components, letting researchers assemble custom memory agents in a "Lego-like" fashion.

The framework natively integrates Group Relative Policy Optimization (GRPO) to fine-tune internal memory-management policies driven by multi-dimensional environmental rewards, and ships out-of-the-box support for recent paradigms including Memory-R1, RMM, and MemAgent. The authors validate MemFactory on the open-source MemAgent architecture using its public training/eval data, reporting relative performance gains over the base models of up to 14.8% on average across evaluation sets. The contribution is framed primarily as standardized, extensible infrastructure that lowers the barrier to entry for memory-driven agent research.

## Key points

- First unified training + inference framework dedicated to memory-augmented agents; modular "Lego-like" composition of the memory lifecycle.
- Decomposes memory into atomic, plug-and-play components: extraction, updating, retrieval.
- Natively integrates GRPO to RL-train internal memory-management policies via multi-dimensional environmental rewards.
- Out-of-the-box support for Memory-R1, RMM, and MemAgent paradigms.
- Validated on open-source MemAgent: relative gains of up to 14.8% over base models on average.
- Explicitly modeled on LLaMA-Factory's unified-fine-tuning philosophy, applied to agent memory.

## Concepts & entities

- [[memory-systems]] / [[long-term-memory]] — the framework's central focus (agent memory lifecycle)
- [[reinforcement-learning]], [[grpo]] (GRPO for memory-policy optimization)
- [[llm-agent]], [[retrieval-augmented-generation]], [[fine-tuning]]
- [[reasoning]]
- MemAgent, Memory-R1, RMM (supported paradigms); LLaMA-Factory (inspiration)
- MemTensor (the authoring team / org)

## References

- [arXiv:2603.29493](https://arxiv.org/abs/2603.29493) — Guo et al., 2026 (v4, 7 Apr 2026).
- Code: github.com/MemTensor/MemFactory.
