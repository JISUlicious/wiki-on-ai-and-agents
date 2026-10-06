---
title: "BabyAGI (README, original and current)"
type: source
created: 2026-10-06
updated: 2026-10-07
sources:
  - babyagi-archive-readme.md
  - babyagi-readme.md
authors:
  - Yohei Nakajima
introduces:
  - "[[babyagi]]"
year: 2023
venue: GitHub (yoheinakajima/babyagi, babyagi_archive)
url: https://github.com/yoheinakajima/babyagi_archive
tags:
  - 2023
  - repo
status: complete
importance: medium
---

# BabyAGI

**Sources**:
- `sources/babyagi-archive-readme.md`: the original 2023 README, now at [babyagi_archive](https://github.com/yoheinakajima/babyagi_archive), a September 2024 snapshot.
- `sources/babyagi-readme.md`: the current README of [yoheinakajima/babyagi](https://github.com/yoheinakajima/babyagi).

**Author**: Yohei Nakajima
**Dates**:
- **2023-03-28**: the original "Task-Driven Autonomous Agent" tweet, per the README.
- **2023-04-03**: repository created, per the GitHub API.
- The current README describes the original as "from March 2023".

## Summary (original, 2023)

[[babyagi|BabyAGI]] is a minimal "AI-powered task management system" of about 140 lines of Python. It uses OpenAI models plus a vector database (Chroma or Weaviate in the later archived README; **Pinecone** in the first version, per `classic/README.md`; see [[babyagi-variants-nakajima-2023]]). Given an **objective** and an initial task, it loops forever:

1. Pull the first task from the task list.
2. An **execution agent** completes the task, using results retrieved from the vector store as context.
3. Store the enriched result in the vector database.
4. A **task-creation agent** writes new tasks from the objective and the last result. A **prioritization agent** reorders the list.

The README warns about runaway API costs from running it continuously. It is a "pared-down version" of Nakajima's tweeted design, and "inspired many projects".

## The 2024 rewrite

The current repository is a different project: "an experimental framework for a self-building autonomous agent", built on **functionz**, a database-backed function graph. The README credits the original with introducing "task planning as a method for developing autonomous agents". It carries a candid caution: the author "has never held a job as a developer" and the code is not meant for production.

## Lineage

- **Built on:** the GPT-4 and ChatGPT API, embedding retrieval (the RAG pattern), and the agent loop ideas of [[react|ReAct]].
- **Contemporaries:** [[autogpt|AutoGPT]] (repo created 2023-03-16), [[generative-agents|Generative Agents]] (April 2023). Lilian Weng's [[llm-powered-autonomous-agents-weng-2023|June 2023 survey]] names BabyAGI among proof-of-concept agents.
- **Fed into:** the planner / executor / prioritizer split, an early [[planning]]-plus-memory agent template; the [[multi-agent]] role decomposition; and later [[self-improving-agent]] ideas, via the 2024 "self-building" rewrite.

## Concepts & entities

[[babyagi]] · [[autogpt]] · [[llm-agent]] · [[planning]] · [[long-term-memory]] · [[multi-agent]]
