---
title: Tiered Memory
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - memgpt-packer-2023.md
status: draft
importance: medium
tags:
  - 2023
---

# Tiered Memory

**Tiered memory** is the general pattern of storing agent memory across hot / warm / cold tiers, where placement is driven by recency, importance, or access frequency. *Hot* is the current prompt context (cheapest to read, hard-capped in size). *Warm* is a session log or recall buffer — accessible in one tool call, e.g. [[memgpt]]'s `recall storage`. *Cold* is archival: vector databases, graph stores, or on-disk files retrieved only when needed. Items are promoted toward hot tiers on access and demoted when stale.

The pattern is explicit in [[memgpt]]'s [[virtual-context]] design and implicit in nearly every production agent: `CLAUDE.md` auto-load behaves as a hot tier, SQLite session logs as warm, and archived `MEMORY.md` files or vector stores as cold.

## Where it appears

- [[memgpt]] — first formalization in an LLM agent setting.
- [[virtual-context]] — the OS-flavored framing of the same tier structure.
- [[memory-management]] — tiering is one axis of the broader memory-management design space (alongside [[ebbinghaus-forgetting|forgetting policy]], consolidation, indexing).

## References

- [[memgpt-packer-2023]]
