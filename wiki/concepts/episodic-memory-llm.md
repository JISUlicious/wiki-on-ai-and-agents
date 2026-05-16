---
title: Episodic Memory (LLM Agents)
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - mirix-wang-2025.md
  - em-llm-fountas-2024.md
  - reflexion-shinn-2023.md
status: draft
importance: medium
tags:
  - 2025
---

# Episodic Memory (LLM Agents)

Memory of specific events with temporal and (often) spatial context, applied to LLM agents. The term comes from cognitive science — [Tulving (1972)](https://en.wikipedia.org/wiki/Episodic_memory) distinguished **episodic memory** (autobiographical events tied to a time and place) from **semantic memory** (general facts decontextualized from when/where they were learned). In LLM-agent memory designs, episodic memory captures *what happened during a session or trajectory*, structured around events rather than free-floating facts.

> [!note] Page slug
> Filed as `episodic-memory-llm` to avoid collision with the broader cognitive-science literature on human episodic memory.

## Where it appears

- [[mirix]] defines an explicit **Episodic** memory type alongside Semantic, Procedural, Resource, and Knowledge Vault, with fields `event_type / summary / details / actor / timestamp`.
- [[reflexion]] uses an **episodic buffer** of past attempts so an agent can reflect on prior trajectories before retrying.
- [[em-llm]] segments the token stream inside the [[kv-cache|KV-cache]] into discrete **events** using Bayesian-surprise boundaries, then retrieves events by similarity plus temporal contiguity.

## References

- [[mirix-wang-2025]]
- [[em-llm-fountas-2024]]
- [[reflexion-shinn-2023]]
