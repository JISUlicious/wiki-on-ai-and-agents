---
title: Memory Evolution
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - a-mem-xu-2025.md
status: draft
importance: medium
tags:
  - 2025
---

# Memory Evolution

The principle that adding a new memory should **update the representations of existing related memories**, not merely append a new record to a store. Introduced by [[a-mem]] ([[a-mem-xu-2025|Xu et al. 2025]]), inspired by the Zettelkasten note-taking method: every memory carries keywords, tags, and links to neighbors, and the arrival of a new memory triggers an LLM-driven rewrite of the surrounding notes — refining their tags, clarifying their content, and re-weaving their links.

This breaks the **append-only** assumption that underlies most prior LLM-agent memory systems: [[mem0]] adds/updates atomic facts but does not propagate to neighbors, and [[memgpt]] pages content in and out of the context window without rewriting older memories.

## Where it appears

- [[a-mem]] — the canonical formulation; each `add(memory)` step performs link construction *and* a propagated rewrite of related notes.
- Contrast with [[mem0]] — atomic per-fact ADD/UPDATE/DELETE/NOOP without neighbor propagation.
- Contrast with [[memgpt]] — hierarchical paging, no representational rewriting.

## References

- [[a-mem-xu-2025]]
