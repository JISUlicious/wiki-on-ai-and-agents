---
title: "Agent Harness for Large Language Model Agents: A Survey"
type: source
created: 2026-06-09
updated: 2026-06-09
sources:
  - agent-harness-survey-meng-2026.md
authors:
  - Qianyu Meng
  - Yanan Wang
  - Liyi Chen
  - Wei Wu
  - Yihang Li
  - Wenyuan Jiang
  - Qimeng Wang
  - Chengqiang Lu
  - Yan Gao
  - Yi Wu
  - Yao Hu
first_author: Qianyu Meng
year: 2026
venue: "Preprints.org"
status: complete
importance: medium
tags:
  - 2026
  - survey
---

# Agent Harness for Large Language Model Agents: A Survey

Surfaced while researching [[agentic-harness-engineering]].

> [!note] Abstract / companion-repo summary only
> The preprints.org PDF and landing-page download endpoints (manuscript 202604.0428, v2/v1/plain) were all blocked by the site's Akamai edge CDN ("Access Denied" / HTTP 403) at fetch time, so full text could **not** be retrieved. This page is reconstructed from the survey's public abstract, its GitHub companion repo (`Gloriaameng/Awesome-Agent-Harness`), and its Hugging Face dataset (`GloriaaaM/LLM-Agent-Harness-Survey`). Treat component-level detail as abstract-level, not full-text, fidelity.

## Summary

A survey arguing that as agent tasks grow longer and more complex, reliability is governed by the infrastructure that wraps the model — the **agent execution harness** — rather than by the base model itself. Its headline claim: *the harness, not the model, is the primary determinant of agent reliability at scale*. The survey formalizes the harness as a first-class architectural object with labeled-transition-system semantics (distinguishing safety from liveness of the execution loop), traces its lineage from software test harnesses through RL environments to modern [[llm-agent|LLM-agent]] infrastructure, and reviews 110+ papers and 23 systems (Claude Code, OpenHands, SWE-agent, MetaGPT, AutoGen, MemGPT, Voyager, PRISM/OpenClaw, AIOS) spanning academic work and production deployments (Stripe, OpenAI, Cursor, METR). It is the third-party survey complement to the wiki's [[code-as-harness]] / [[agentic-harness-engineering]] cluster.

## Key points

- **Six-component formalization** `H = (E, T, C, S, L, V)`: Execution loop, Tool registry, Context manager, State store, Lifecycle hooks, and eValuation interface — a governable decomposition of the model-external scaffold, parallel to L2 of the [[agent-three-layer-model]].
- **Harness Completeness Matrix**: a structured assessment mapping which of the six components each system implements, enabling cross-system comparison. Finding: production-grade systems converge on full `ETCSLV`, while research prototypes often implement only 2-3 components.
- **Historical taxonomy** (~1997-2026): the harness concept is traced from software test harnesses, through RL training environments, to today's LLM-agent runtimes.
- **Optimization / self-evolution patterns surveyed**: skill accumulation (Voyager, cf. [[agent-skills]] / [[skill-optimization]]), reflection-based correction (Reflexion), self-evolving agents (Evo-Memory, Self-RAG; cf. [[self-improving-agent]]), [[memory-management|memory]] optimization (MemoryOS, Mem0), planning augmentation (Tree of Thoughts, LATS), and schema-first tool APIs that reduce interface misuse.
- **Nine open challenges**: security & sandboxing, evaluation & benchmarking, protocol standardization (cf. [[mcp]]), runtime context management, tool use & registry, memory architecture, planning & reasoning, [[multi-agent|multi-agent coordination]], and compute economics.

## Concepts & entities

- Concepts: [[agentic-harness-engineering]], [[code-as-harness]], [[self-improving-agent]], [[llm-agent]], [[agent-skills]], [[skill-optimization]], [[memory-management]], [[multi-agent]], [[mcp]], [[agent-three-layer-model]]

## References

- Source note: [[agent-harness-survey-meng-2026]] (provenance + reconstructed notes)
- Landing page: https://www.preprints.org/manuscript/202604.0428 (DOI 10.20944/preprints202604.0428.v2)
- Companion repo: https://github.com/Gloriaameng/Awesome-Agent-Harness
- Companion dataset: https://huggingface.co/datasets/GloriaaaM/LLM-Agent-Harness-Survey
