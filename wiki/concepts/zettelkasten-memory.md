---
title: Zettelkasten Memory
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

The Zettelkasten ("slip-box") is a note-taking philosophy originated by sociologist Niklas Luhmann, in which each atomic note carries keywords, tags, and explicit links to neighboring notes — and adding a new note triggers updates to the surrounding network. The system was popularized in knowledge-management circles as a way to grow an emergent, interlinked second brain rather than a flat archive. Applied to LLM agents, the Zettelkasten metaphor recasts agent memory as a graph of structured notes that *evolve* when new evidence arrives: each new memory carries metadata (keywords, tags, context) and its insertion prompts the agent to revise links and content of neighboring memories. This is the architectural inspiration behind [[a-mem]] (Xu 2025), which treats memory evolution as a first-class operation rather than a side effect.

## Where it appears

- [[a-mem]] — Direct architectural inspiration; each memory note carries keywords/tags/links/context and triggers neighbor updates.
- [[memory-evolution]] — The Zettelkasten-style "update neighbors on insert" behavior generalized as an LLM memory operation.
- [[memory-management]] — One of several philosophical ancestors of structured agent memory.

## References

- [[a-mem-xu-2025]] — A-MEM: Agentic Memory for LLM Agents (Xu 2025).
