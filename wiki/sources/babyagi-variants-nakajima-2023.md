---
title: "BabyAGI classic and the Baby<animal>AGI variants (2023)"
type: source
created: 2026-10-07
updated: 2026-10-07
sources:
  - babyagi-archive-classic-README.md
  - babyagi-archive-classic-ABOUT.md
  - babyagi-archive-variants.md
  - babyagi-archive-docs-inspired-projects.md
authors:
  - Yohei Nakajima
year: 2023
venue: GitHub (yoheinakajima/babyagi_archive)
url: https://github.com/yoheinakajima/babyagi_archive/tree/main/classic
tags:
  - 2023
  - repo
status: complete
importance: medium
---

# BabyAGI classic and the Baby<animal>AGI variants

**Sources** (all from `yoheinakajima/babyagi_archive`):
- `classic/README.md`: the **first** README, which uses Pinecone;
- `classic/ABOUT.md`;
- the opening lines of `BabyBeeAGI.py`, `BabyCatAGI.py` and `BabyDeerAGI.py`;
- `babyfoxagi/README.md`;
- `docs/inspired-projects.md`.

**Dating method:** each variant's README links its announcement tweet. Tweet IDs (Twitter "snowflake" IDs) encode the post time, which gives the dates below in UTC. GitHub commit dates in the archive are later file moves, so they are not used.

## The classic script (March 2023)

- **Stack:** OpenAI plus **Pinecone**, then the only vector store. Chroma and Weaviate support came later, per the [[babyagi-nakajima-2023|later README]].
- **Loop:** an infinite execute → enrich and store → create tasks → reprioritize cycle.
- **Size:** 140 lines (13 comments, 22 blank, 105 code).
- **Origin:** a pared-down version of the "Task-Driven Autonomous Agent" tweet.
  - The README dates the tweet Mar 28, 2023; the tweet ID decodes to **2023-03-29 UTC**, presumably the same post seen in a US time zone.
  - "The name of the repo came up in the reaction to the original autonomous agent — the author does not mean to imply that this is AGI."
  - The author describes himself as a VC.

## The variants (Baby<animal>AGI)

| Variant | Announced (UTC, from the tweet ID) | What changed (per its header or README) |
|---|---|---|
| **BabyBeeAGI** | 2023-04-30 | First mod; "requires GPT-4, it's very slow, and often errors out"; optional **web search** (SerpAPI) |
| **BabyCatAGI** | 2023-05-13 | Continued the line; web-search tool; example objective "Research experts at scaling NextJS…" |
| **BabyDeerAGI** | 2023-06-07 | Optional **user-input tool**; example "Research recent AI news and write a poem…" |
| **BabyElfAGI** | 2023-07-10 | Linked from the Fox README; the archive README for Elf is missing (404) |
| **BabyFoxAGI** | 2023-09-01 | "The 5th mod": **self-improvement** of task-list building (the "FOXY method", tweet 2023-07-31); a **chat UI** that separates chat from the task list, where a request can run one *skill* quickly or chain skills; parallel tasks removed after problems |

Every variant header says the same thing: shared "as a framework to build on top of… to facilitate discussion", "NOT for people who are looking for a complete solution".

## Inspired projects

The archive's curated list includes:
- BabyAGI on Slack;
- **AgentGPT**, which deploys agents in the browser;
- AgentLLM, with a browser-native LLM;
- BabyAGI-asi, which runs arbitrary Python;
- ai-legion (TypeScript);
- a Swift port;
- a ChatGPT plugin that runs BabyAGI;
- Cognosys;
- Godmode;
- and others.

## Why it matters for the wiki

In five months the variants trace the agent design moves of 2023:
1. a plain task loop;
2. tools (web search);
3. a human-in-the-loop tool;
4. reusable **skills** and their chaining;
5. self-improvement of planning.

These are early, informal versions of what later became [[tool-use]], [[agent-skills]] (2025) and [[self-improving-agent|self-improving agents]].

## Concepts & entities

[[babyagi]] · [[autogpt]] · [[llm-agent]] · [[planning]] · [[agent-skills]] · [[self-improving-agent]] · [[tool-use]]
