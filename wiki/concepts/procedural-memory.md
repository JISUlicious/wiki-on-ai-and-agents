---
title: Procedural Memory
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - voyager-wang-2023.md
  - mirix-wang-2025.md
status: draft
importance: medium
tags:
  - 2023
---

# Procedural Memory

**Procedural memory** is memory of *how to do things* rather than *what is true*. In cognitive science it covers skills, habits, and motor sequences — knowledge expressed by doing. In LLM agents, the analogue is a library of executable code skills (the [[voyager]] [[skill-library]]), a collection of `SKILL.md` packages, or learned tool-use routines that can be retrieved and invoked rather than re-derived each time. It is distinct from declarative memory, which splits into semantic (facts) and episodic (events) memory.

The [[mirix]] paper ([[mirix-wang-2025|Wang et al. 2025]]) makes procedural memory one of its six explicit memory roles, alongside core, episodic, semantic, resource, and knowledge-vault memory.

## Where it appears

- [[voyager]] — Minecraft skill library; each skill is JavaScript code keyed by an NL description embedding.
- [[mirix]] — one of six typed memory roles in the six-memory architecture.
- [[skill-library]] — the operational data structure that implements procedural memory in practice.

## References

- [[voyager-wang-2023]]
- [[mirix-wang-2025]]
