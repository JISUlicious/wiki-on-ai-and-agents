---
title: Multi-Agent Memory
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - mirix-wang-2025.md
status: draft
importance: medium
tags:
  - 2025
---

Multi-agent memory is an architectural pattern in which several sub-agents each specialize in a different *type* of memory (and a different set of operations on it), coordinated by a meta-controller that routes incoming information and queries to the appropriate specialist. Instead of one monolithic memory store, the system maintains parallel, semantically distinct stores managed by dedicated agents. The most explicit instance is [[mirix]] (Wang & Chen 2025), which decomposes memory into six types — Core, Episodic, Semantic, Procedural, Resource, and Knowledge Vault — each managed by its own agent under a meta-controller. The design borrows from cognitive-science models that distinguish episodic/semantic/procedural memory in humans.

## Where it appears

- [[mirix]] — Direct introduction; six memory types, one agent per type, coordinated by a meta-controller.
- [[memory-management]] — Multi-agent memory is one architectural style in the broader memory design space.
- [[llm-agent]] — The host substrate for multi-agent memory systems.

## References

- [[mirix-wang-2025]] — MIRIX: Multi-Agent Memory System for LLM-Based Agents (Wang & Chen 2025).
