---
title: BabyAGI
type: entity
created: 2026-10-06
updated: 2026-10-06
sources:
  - babyagi-archive-readme.md
  - babyagi-readme.md
release_date: 2023-03-28
status: complete
importance: medium
tags:
  - product
---

# BabyAGI

An open-source "task-driven autonomous agent" by Yohei Nakajima.
- **2023-03-28:** described in a tweet.
- **2023-04-03:** GitHub repository created.

It is about 140 lines of Python: OpenAI models plus a vector store, in an endless loop of **execute → store result → create new tasks → reprioritize**. Each step is handled by its own LLM "agent" (execution, task creation, prioritization). With [[autogpt|AutoGPT]] (repo created 2023-03-16), it was one of the two viral agent demos of spring 2023.

- **Significance:** it introduced "task planning as a method for developing autonomous agents" (its own README's later summary). It is an early template for [[planning]] plus [[long-term-memory]] agents, and for splitting an agent into LLM roles ([[multi-agent]]).
- **Reception:** [[lilian-weng|Lilian Weng]]'s June 2023 survey lists it among proof-of-concept agents ([[llm-powered-autonomous-agents-weng-2023]]).
- **2024:** the original was archived (`babyagi_archive`). The repository now holds a different experimental "self-building" agent framework built on *functionz*.

## References

- [[babyagi-nakajima-2023]]
- [[llm-powered-autonomous-agents-weng-2023]]
