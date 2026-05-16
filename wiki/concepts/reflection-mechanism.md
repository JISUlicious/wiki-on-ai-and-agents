---
title: Reflection Mechanism
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - generative-agents-park-2023.md
status: draft
importance: medium
tags:
  - 2023
---

# Reflection Mechanism

The **reflection mechanism** is the periodic synthesis of higher-level memories from many low-level observations, introduced by [[generative-agents]] ([[generative-agents-park-2023|Park et al. 2023]]). When accumulated importance scores cross a threshold, the agent prompts itself — roughly *"Given the recent [[memory-stream]], what 3-5 high-level insights can be drawn?"* — and writes the resulting insights back into the stream as new, higher-abstraction entries. Over time this raises the conceptual level at which the agent reasons about itself and others.

This is the [[memory-management]] *consolidate* operation in its canonical LLM form. Distinguish from [[reflexion]] ([[reflexion-shinn-2023|Shinn et al. 2023]]), which uses self-critique across *retry attempts* on a task — a different verb that happens to share the name.

## Where it appears

- [[generative-agents]] — original implementation in the Smallville agents.
- [[mirix]] — generalizes consolidation across six memory types.
- [[memory-management]] — reflection is one of the named lifecycle operations (write, retrieve, **consolidate**, evict).

## References

- [[generative-agents-park-2023]]
