---
title: ChatGPT plugins
type: entity
created: 2026-10-06
updated: 2026-10-06
sources:
  - chatgpt-plugins-openai-2023.md
release_date: 2023-03-23
status: complete
importance: medium
tags:
  - product
---

# ChatGPT plugins

[[openai|OpenAI]]'s first tool platform for [[chatgpt|ChatGPT]], announced **2023-03-23** as a waitlisted alpha.

- **Mechanism:** a third-party developer exposes an API, describes it with an **OpenAPI spec** and a **manifest** (with a model-facing name and description), and ChatGPT inserts the documentation of the enabled plugins into the prompt. The model decides when to call them.
- **Plugins hosted by OpenAI:** **browsing**, a descendant of [[webgpt|WebGPT]] (GET-only, Bing API, cites sources); and **code interpreter** (sandboxed Python).
- **Open source:** a **retrieval plugin**.
- **Launch partners:** Expedia, Instacart, KAYAK, Klarna, OpenTable, Wolfram, Zapier and others.
- **Safety:** red-teamers found that plugins without safeguards could enable "sophisticated [[prompt-injection|prompt injection]]", spam and fraud.

## Lineage

- **Built on:** [[webgpt|WebGPT]], [[toolformer|Toolformer]] and the [[react|ReAct]] loop.
- **Successors:**
  - [[function-calling]] (June 2023), with tool schemas in the API;
  - GPTs and Actions (Nov 2023), which replaced plugins; plugins were retired in 2024 (both general knowledge).
- **The predicted standard:** OpenAI called plugins "an early attempt" at an open standard for AI-facing interfaces. That standard later arrived from Anthropic as [[mcp|MCP]] (Nov 2024).
- **Not the same as Claude Code plugins:** those are bundles of skills, hooks and MCP servers for a coding agent (Oct 2025; see [[claude-code-changelog-2026]]).

## References

- [[chatgpt-plugins-openai-2023]]
- [[llm-powered-autonomous-agents-weng-2023]]
