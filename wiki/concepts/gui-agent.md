---
title: GUI Agent
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - ui-tars-2-wang-2025.md
status: draft
importance: high
tags:
  - agent
---

A **GUI agent** is an [[llm-agent]] specialized for navigating graphical user interfaces: web pages, desktop applications, and mobile screens. It is a subset of the broader [[computer-use-agent]] category, distinguished by its reliance on visual / DOM-level perception and discrete click/type/scroll actions rather than arbitrary shell or API tool calls.

- Perception: screenshots + optional accessibility tree / DOM
- Actions: click, type, scroll, drag, key combos at element or pixel level
- Common grounding primitive: [[set-of-mark]]
- Benchmarks: [[osworld]], WebArena, Mind2Web

Related: [[computer-use-agent]], [[ui-tars-2]], [[magma]], [[osworld]], [[vision-language-action-model]]

## References
- [[ui-tars-2-wang-2025]]
- [[magma-yang-2025]]
