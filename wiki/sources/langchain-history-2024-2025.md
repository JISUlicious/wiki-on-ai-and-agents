---
title: "LangChain v0.1.0, LangGraph, and LangChain/LangGraph 1.0 (LangChain blog)"
type: source
created: 2026-10-07
updated: 2026-10-07
sources:
  - langchain-v0-1-0-2024.md
  - langgraph-launch-2024.md
  - langchain-langgraph-1-0-2025.md
authors:
  - LangChain Team
year: 2025
venue: LangChain blog
url: https://blog.langchain.com/
tags:
  - blog
status: complete
importance: medium
---

# LangChain's evolution: v0.1 → LangGraph → 1.0

**Sources**: three posts from LangChain's own blog (fetched 2026-10-07):
- `langchain-v0-1-0-2024.md`: [LangChain v0.1.0](https://blog.langchain.com/langchain-v0-1-0/), **2024-01-08**
- `langgraph-launch-2024.md`: [LangGraph](https://blog.langchain.com/langgraph/), **2024-01-17**
- `langchain-langgraph-1-0-2025.md`: [LangChain and LangGraph Agent Frameworks Reach v1.0 Milestones](https://blog.langchain.com/langchain-langgraph-1dot0/), **2025-10-22**, by Sydney Runkle and the LangChain OSS team

These are vendor posts.

## v0.1.0: the first stable release (Jan 2024)

- LangChain "has been around for a little over a year" and has become "the default framework for building LLM applications" (their claim).
- **Restructuring:**
  - `langchain-core` holds the abstractions and interfaces;
  - all third-party integrations move to `langchain-community` or standalone partner packages, "almost 700 integrations".
- **Versioning:** semantic-style from here on. Breaking API changes bump the minor version.
- The release also stresses observability (LangSmith), streaming, output parsing, retrieval, and the LangChain Expression Language (**LCEL**) for composing chains.

## LangGraph: cycles for agents (Jan 2024)

- **The problem:** LCEL chains are **DAGs**, but agents need **cycles**, "running an LLM in a for-loop".
- **The basic loop:** "the loop that powers our core AgentExecutor… the same logic that caused projects like AutoGPT to rise in prominence":
  1. call the LLM to decide actions or a response;
  2. take the actions;
  3. repeat.
- **LangGraph** builds cyclical graphs with state on top of LangChain, so developers can **constrain** the loop: force a tool call first, add human-in-the-loop, manage state. It sits between a fully autonomous loop and a fixed chain.

## 1.0 (Oct 2025)

The first major versions, with "no breaking changes until 2.0". The two frameworks have different roles:
- **LangChain 1.0**: "the fastest way to build an AI agent".
  - **`create_agent`** wraps the core tool-calling loop. It is built on the LangGraph runtime and modelled on `langgraph.prebuilt.create_react_agent`, which "has been used in production for a year".
  - **Middleware**: "a set of hooks that allow you to customize behavior in the agent loop". The built-in middleware covers:
    - **human-in-the-loop** approval of tool calls;
    - **summarization** when nearing context limits, i.e. compaction;
    - **PII redaction**.
  - **Standard content blocks**: a provider-agnostic format for model outputs.
  - The admitted feedback: "LangChain's abstractions were sometimes too heavy… developers wanted more control over the agent loop."
- **LangGraph 1.0**: a lower-level runtime for "production-grade, long running agents", with durable state.

## Why it matters for the wiki

- **The industry's arc in one framework:** chains ([[prompt-chaining]]) → cyclical agent graphs ([[react|ReAct]]-style loops) → one agent loop customized through hooks.
- **Convergence with Claude Code:** LangChain's "middleware" is the same idea as [[hooks-in-modern-agents|hooks]]. The built-in summarization middleware is [[context-engineering|compaction]].

## Concepts & entities

[[langchain]] · [[prompt-chaining]] · [[llm-agent]] · [[react]] · [[hooks-in-modern-agents]] · [[context-engineering]] · [[autogpt]]
