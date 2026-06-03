---
title: "Memory Transfer Learning: How Memories are Transferred Across Domains in Coding Agents"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - memory-transfer-learning-coding-agents-kim-2026.md
arxiv_id: "2604.14004"
authors:
  - Kangsan Kim
  - Minki Kang
  - Taeil Kim
  - Yanlai Yang
  - Mengye Ren
  - Sung Ju Hwang
first_author: Kangsan Kim
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Memory Transfer Learning: How Memories are Transferred Across Domains in Coding Agents

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

This paper studies **Memory Transfer Learning (MTL)** for self-evolving coding agents: instead of restricting memory to a single homogeneous task domain, it harnesses a unified memory pool drawn from heterogeneous domains that share infrastructural foundations (runtime environments, programming languages). Across 6 coding benchmarks and four memory representations (from concrete traces to abstract insights), cross-domain memory improves average performance by 3.7%, mainly by transferring meta-knowledge such as validation routines rather than task-specific code. A central finding is that **abstraction dictates transferability**: high-level insights generalize, while low-level traces often cause negative transfer.

## Key points

- Frames the limitation of prior memory-based self-evolution: memory utilization is siloed within single task domains, ignoring shared cross-domain infrastructure.
- Builds a unified memory pool from heterogeneous coding domains and evaluates four memory representations ranging from concrete traces to abstract insights.
- Cross-domain memory yields +3.7% average performance, primarily by transferring meta-knowledge (e.g., validation routines), not task-specific code.
- **Abstraction is the key control variable**: high-level insights generalize well; low-level traces induce negative transfer through excessive specificity.
- Transfer effectiveness scales with memory-pool size, and memory can transfer even between different models.
- Establishes empirical design principles for expanding agent memory beyond single-domain silos; from KAIST, NYU, and DeepAuto.ai.

## Concepts & entities

- [[memory-management]]
- [[memory-construction]]
- [[self-improving-agent]]
- [[experiential-learning]]
- [[procedural-memory]]
- [[swe-bench]]

## References

- [arXiv:2604.14004](https://arxiv.org/abs/2604.14004)
- Source text: `sources/memory-transfer-learning-coding-agents-kim-2026.md`
