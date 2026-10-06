---
title: "LangChain README (2026)"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - langchain-readme.md
introduces:
  - "[[langchain]]"
year: 2026
venue: GitHub (langchain-ai/langchain)
url: https://github.com/langchain-ai/langchain
tags:
  - 2026
  - repo
status: complete
importance: low
---

# LangChain README (2026)

**Source**: `sources/langchain-readme.md` ([langchain-ai/langchain](https://github.com/langchain-ai/langchain) README, fetched 2026-10-06)
**Origin date**: the repository was **created 2022-10-17**, per the GitHub API, recorded in the source header. Harrison Chase as creator is general knowledge; the README does not name him.

## Summary

The current README describes [[langchain|LangChain]] as "**the agent engineering platform**": "a framework for building agents and LLM-powered applications" that helps "chain together interoperable components and third-party integrations".

The 2026 ecosystem it lists:
- **LangGraph**: low-level orchestration for controllable agent workflows.
- **Deep Agents**: a higher-level package with built-in planning, subagents and file-system use.
- **LangSmith**: tracing, evaluation and deployment.
- **LangChain.js**: the JavaScript version.

The quick-start example calls `init_chat_model("openai:gpt-5.5")`.

## Lineage

- **2022-10:** LangChain launches about a month before ChatGPT. The name reflects its first abstraction, a *chain* of LLM calls: the [[prompt-chaining]] idea that [[ai-chains-wu-2021|AI Chains]] (Wu et al., Oct 2021) had studied.
- **2023:** LangChain's agents implement [[react|ReAct]]-style tool loops and power many [[autogpt|AutoGPT]]- and [[babyagi|BabyAGI]]-era projects.
- **2024–2026:**
  - It moves toward graph orchestration (LangGraph) and "agent engineering".
  - Anthropic's [[building-effective-agents-anthropic-2024|Building effective agents]] (Dec 2024) advises starting with direct API calls before using frameworks.
  - LangChain's 2026 agent-evaluation material is already in the wiki: [[agent-evaluation-readiness-checklist-langchain-2026]].

> [!note] Limits of this source
> The README is promotional and current-state only; it gives no history. The October 2022 date comes from repository metadata.

## Concepts & entities

[[langchain]] · [[prompt-chaining]] · [[llm-agent]] · [[react]] · [[multi-agent]] · [[subagents]]
