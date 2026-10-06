---
title: LangChain
type: entity
created: 2026-10-06
updated: 2026-10-06
sources:
  - langchain-readme.md
release_date: 2022-10-17
status: draft
importance: medium
tags:
  - product
---

# LangChain

An open-source framework for LLM applications and agents. The GitHub repository `langchain-ai/langchain` was **created 2022-10-17**, about six weeks before ChatGPT. It was created by Harrison Chase (general knowledge; the README does not name him).

- **Origin:** its name and first abstraction are *chains* of LLM calls, the [[prompt-chaining]] idea studied in [[ai-chains-wu-2021|AI Chains]] (Oct 2021). It soon added [[react|ReAct]]-style agents and integrations for retrieval and tools, and many 2023 agent projects were built on it.
- **2026 positioning:** "the agent engineering platform". Its ecosystem:
  - **LangGraph**: controllable agent workflows as graphs;
  - **Deep Agents**: built-in planning, [[subagents]] and file-system use;
  - **LangSmith**: tracing, evaluation and deployment.
- **The counterpoint:** Anthropic's [[building-effective-agents-anthropic-2024|Building effective agents]] advises starting with direct API calls, because framework abstractions "can obscure the underlying prompts and responses".
- **In the wiki:** LangChain's agent-evaluation guide, [[agent-evaluation-readiness-checklist-langchain-2026]].

## References

- [[langchain-readme-2026]]
- [[ai-chains-wu-2021]]
