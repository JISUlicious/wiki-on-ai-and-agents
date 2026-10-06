---
title: LangChain
type: entity
created: 2026-10-06
updated: 2026-10-07
sources:
  - langchain-readme.md
  - langchain-v0-1-0-2024.md
  - langgraph-launch-2024.md
  - langchain-langgraph-1-0-2025.md
release_date: 2022-10-17
status: complete
importance: medium
tags:
  - product
---

# LangChain

An open-source framework (Python and JavaScript) for building LLM applications and agents. The GitHub repo `langchain-ai/langchain` was **created 2022-10-17**, about six weeks before ChatGPT. The creator, Harrison Chase, is general knowledge; the README doesn't name him.

It positions itself in 2026 as "the agent engineering platform". Ecosystem:
- **LangGraph**: runtime and orchestration;
- **LangSmith**: tracing and evaluation;
- **Deep Agents**: a higher-level agent with planning, [[subagents]] and a file system.

## Timeline

| Date | Milestone | What it shows |
|---|---|---|
| **2022-10-17** | Repo created | *Chains* of LLM calls: the [[prompt-chaining]] idea studied in [[ai-chains-wu-2021\|AI Chains]] (Oct 2021) |
| 2023 | Agents (`AgentExecutor`, ReAct-style), retrieval and tool integrations | Many [[autogpt\|AutoGPT]]- and [[babyagi\|BabyAGI]]-era projects built on it |
| **2024-01-08** | **v0.1.0**: first stable release | Split into `langchain-core` and `langchain-community` (about 700 integrations); LCEL for composing chains; LangSmith observability |
| **2024-01-17** | **LangGraph** | Chains are DAGs, but agents need **cycles**. Stateful cyclical graphs let developers *constrain* the agent loop: force tool calls, add human approval |
| **2025-10-22** | **LangChain 1.0 and LangGraph 1.0** | `create_agent` built on the LangGraph runtime. **Middleware** hooks in the agent loop: human-in-the-loop, summarization (compaction), PII redaction. Standard content blocks. No breaking changes until 2.0 |

## What its arc shows

LangChain's history follows the field's:
1. **Chains** (fixed pipelines, 2022).
2. **Agents** (open loops, 2023).
3. **Graphs** that put structure back into the loop (2024).
4. **One agent loop customized through hooks** (2025).

The 1.0 post admits the abstractions had been "sometimes too heavy" and that developers "wanted more control over the agent loop". Anthropic's [[building-effective-agents-anthropic-2024|Building effective agents]] (Dec 2024) had made the same criticism of frameworks generally: start with direct API calls.

The 2025 *middleware* design converges with coding-agent harnesses: compare [[hooks-in-modern-agents|hooks]] and [[claude-code-mods|mods]] in Claude Code.

## In the wiki

- [[agent-evaluation-readiness-checklist-langchain-2026]] and [[langsmith-evaluation-docs]] cover evaluation.

## References

- [[langchain-readme-2026]]
- [[langchain-history-2024-2025]]
- [[ai-chains-wu-2021]]
