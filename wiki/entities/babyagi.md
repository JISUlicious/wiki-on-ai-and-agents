---
title: BabyAGI
type: entity
created: 2026-10-06
updated: 2026-10-07
sources:
  - babyagi-archive-readme.md
  - babyagi-readme.md
  - babyagi-archive-classic-README.md
  - babyagi-archive-variants.md
  - babyagi-archive-docs-inspired-projects.md
founders:
  - Yohei Nakajima
release_date: 2023-03-28
status: complete
importance: medium
tags:
  - product
---

# BabyAGI

An open-source "task-driven autonomous agent" by Yohei Nakajima, a venture capitalist by his own description. It is one of the two viral agent demos of spring 2023, alongside [[autogpt|AutoGPT]] (repo created 2023-03-16).

The name came from reactions to the original tweet. The README says the author "does not mean to imply that this is AGI."

## Timeline

| Date (UTC) | Event | Source |
|---|---|---|
| 2023-03-28/29 | "Task-Driven Autonomous Agent" tweet. The README says Mar 28; the tweet ID decodes to Mar 29 UTC. | classic README |
| 2023-04-03 | GitHub repository created | GitHub API |
| 2023-04-30 | **BabyBeeAGI**: first variant; needs GPT-4; optional web search | variant header + tweet ID |
| 2023-05-13 | **BabyCatAGI** | tweet ID |
| 2023-06-07 | **BabyDeerAGI**: adds a user-input tool | tweet ID |
| 2023-06-23 | Named in [[lilian-weng\|Lilian Weng]]'s agents survey | [[llm-powered-autonomous-agents-weng-2023]] |
| 2023-07-10 | **BabyElfAGI** | tweet ID |
| 2023-09-01 | **BabyFoxAGI**: self-improving task-list building (the "FOXY method"); a chat UI that runs one skill or chains skills | Fox README + tweet ID |
| 2024-09 | Original archived to `babyagi_archive`; the repo becomes a new "self-building" agent framework (*functionz*) | current README |

## The design

About 140 lines of Python: OpenAI models plus a **Pinecone** vector index. Chroma and Weaviate were supported later. The loop runs forever:
1. take the first task;
2. an **execution agent** completes it, with context retrieved from earlier results;
3. store the result in the vector index;
4. a **task-creation agent** proposes new tasks toward the **objective**;
5. a **prioritization agent** reorders the queue.

Each "agent" is a separately prompted LLM call, an early role split ([[multi-agent]]). Retrieval over past results is a rudimentary [[long-term-memory]].

## Significance

- **A template:** objective → task list → execute → reflect and re-plan became a standard early [[planning]] pattern. Its own later summary says it introduced "task planning as a method for developing autonomous agents".
- **An evolutionary record:** in five months the variants moved from a bare loop to tools (web search), a human in the loop, reusable **skills** and chaining, and **self-improvement** of the planner. These are informal precursors of [[tool-use]], [[agent-skills]] and the [[self-improving-agent]] work of 2025–26.
- **A community:** a long tail of derived projects, including AgentGPT, Cognosys, Godmode, Slack and Swift ports, and a ChatGPT plugin.
- **Known limits, stated by the author:** slow, "often errors out", high API cost when left running, and "NOT for people who are looking for a complete solution". Parallel task execution was dropped in BabyFoxAGI after problems.

## References

- [[babyagi-nakajima-2023]]
- [[babyagi-variants-nakajima-2023]]
- [[llm-powered-autonomous-agents-weng-2023]]
