---
title: Virtual Context
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

# Virtual Context

**Virtual context** is [[memgpt]]'s OS-inspired abstraction over the LLM's finite prompt window ([[memgpt-packer-2023|Packer et al. 2023]]). The window is treated as "main memory" (RAM), and the agent emits function calls to page additional content in from `recall storage` (warm — recent message history) or `archival storage` (cold — large vector-indexed document store). Crucially the LLM itself acts as the OS scheduler over its own context: it decides when to evict, when to summarize, and when to fetch — there is no external policy.

The name borrows directly from virtual memory in operating systems: a small fast tier backed by larger slower tiers, with explicit paging operations exposed as a tool API.

## Where it appears

- [[memgpt]] — original system; pioneered the OS analogy for LLM context.
- [[tiered-memory]] — the broader pattern virtual context instantiates.
- [[memory-management]] — virtual context is one named approach to the read/write/evict lifecycle.

## References

- [[memgpt-packer-2023]]
