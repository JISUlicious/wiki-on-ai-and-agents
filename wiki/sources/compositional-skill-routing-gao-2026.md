---
title: "Compositional Skill Routing for LLM Agents: Decompose, Retrieve, and Compose"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - compositional-skill-routing-gao-2026.md
arxiv_id: "2606.18051"
authors:
  - Xueping Gao
first_author: Xueping Gao
year: 2026
venue: arXiv preprint
tags:
  - 2026
status: complete
importance: medium
---

# Compositional Skill Routing for LLM Agents: Decompose, Retrieve, and Compose

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-14–06-21).

## Summary

This paper formalizes **Compositional Skill Routing**: given a complex user query and a large library of reusable skills (tool specifications), the agent must decompose the query into atomic sub-tasks, retrieve the appropriate skill for *each* sub-task, and compose an executable plan — selecting and sequencing *multiple* skills rather than picking one ([[agent-skills]], [[skill-acquisition]]). The proposed **SkillWeaver** framework follows a decompose→retrieve→compose pipeline combining an LLM task decomposer, a bi-encoder skill retriever with FAISS indexing ([[retrieval-augmented-generation]]), and a dependency-aware DAG planner.

To evaluate, the authors build **CompSkillBench** ([[agent-benchmark]]), 300 compositional queries over 2,209 real [[mcp]]-server skills across 24 functional categories from the public MCP ecosystem. They find decomposition quality is the primary bottleneck — standard LLM decomposition reaches only 34.2% step-level category recall — and introduce **Iterative Skill-Aware Decomposition (SAD)**, a retrieval-augmented feedback loop that aligns decomposition with the available skill inventory, lifting decomposition accuracy from **51.0% to 67.7%** in a single iteration.

## Key points

- **Problem formalization:** compositional skill routing = decompose query → retrieve one skill per sub-task → compose a dependency-aware executable plan.
- **SkillWeaver pipeline:** LLM decomposer + bi-encoder retriever over FAISS + DAG planner that respects sub-task dependencies.
- **CompSkillBench:** 300 compositional queries over **2,209 real MCP-server skills** spanning 24 functional categories from the public MCP ecosystem.
- **Decomposition is the bottleneck:** naive LLM decomposition hits only 34.2% step-level category recall.
- **Iterative Skill-Aware Decomposition (SAD)** improves decomposition accuracy **51.0% → 67.7%** (+32.7% relative, Wilcoxon p < 10⁻⁶) in one iteration; correct granularity raises CatR@1 from 34% to 41%.
- **Efficiency + transfer:** SkillWeaver cuts context-window consumption by >99% and shows +35.6% relative gain even when target categories are absent from the retrieval pool.

## Concepts & entities

- [[agent-skills]], [[skill-acquisition]] — composing/selecting reusable skills from large libraries.
- [[mcp]] — skills are sourced from real MCP-server tool specifications.
- [[retrieval-augmented-generation]] — bi-encoder + FAISS retrieval of skills per sub-task.
- [[agent-benchmark]] — CompSkillBench evaluation suite.

## References

- [arXiv:2606.18051](https://arxiv.org/abs/2606.18051)
- Source PDF: `compositional-skill-routing-gao-2026.pdf`
