---
title: "StructuredAgent: Planning with AND/OR Trees for Long-Horizon Web Tasks"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - structuredagent-andor-tree-planning-lobo-2026.pdf
arxiv_id: "2603.05294"
authors:
  - Elita A. Lobo
  - Xu Chen
  - Jingjing Meng
  - Nan Xi
  - Yang Jiao
  - Chirag Agarwal
  - Yair Zick
  - Yan Gao
first_author: Elita A. Lobo
year: 2026
tags: [2026]
status: complete
importance: medium
---

# StructuredAgent: Planning with AND/OR Trees for Long-Horizon Web Tasks

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

StructuredAgent is a hierarchical planning framework for LLM-based web agents that struggle on long-horizon tasks because of limited in-context memory, weak planning, poor error recovery, and greedy premature termination. It interleaves planning and execution by dynamically constructing ordered AND/OR trees — AND nodes for subgoals, OR nodes for alternative strategies, and atomic web actions at the leaves — so the agent can reason about fallbacks and produce interpretable plans.

Crucially, the framework (not the LLM) constructs and maintains the planning tree, invoking the model only for local operations like node expansion or repair. A modified greedy depth-first search drives tree expansion with dynamic plan revision and error back-propagation, while a structured memory module tracks candidate entities and the constraints they satisfy to improve constraint satisfaction on information-seeking tasks.

## Key points

- Decomposes tasks into AND (subgoal) / OR (alternative-strategy) trees plus atomic actions, separating plan management (framework) from local reasoning (LLM).
- Supports dynamic plan revision and error back-propagation, enabling pruning/repair as new information arrives rather than committing to a sequential plan upfront.
- A structured memory module tracks candidate entities and their satisfied constraints, addressing the constraint-satisfaction failures of greedy agents.
- Produces interpretable hierarchical plans that ease debugging and human intervention.
- Evaluated on WebVoyager, WebArena, and a custom shopping benchmark, improving over standard LLM-based web agents on long-horizon browsing.
- Framed by cognitive-science heuristic search / means-ends analysis (Newell et al.).

## Concepts & entities

- [[llm-agent]]
- [[gui-agent]]
- [[computer-use-agent]]
- [[tree-of-thoughts]]
- [[memory-management]]
- [[inference-time-compute]]
- [[agent-benchmark]]

## References

- [arXiv:2603.05294](https://arxiv.org/abs/2603.05294)
- NeurIPS 2025 Workshop on Efficient Reasoning
