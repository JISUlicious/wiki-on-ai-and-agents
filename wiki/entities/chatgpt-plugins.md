---
title: ChatGPT plugins
type: entity
created: 2026-10-06
updated: 2026-10-07
sources:
  - chatgpt-plugins-openai-2023.md
  - function-calling-api-updates-openai-2023.md
  - introducing-gpts-openai-2023.md
  - chatgpt-plugins-wind-down-2024.md
release_date: 2023-03-23
status: complete
importance: medium
tags:
  - product
---

# ChatGPT plugins

[[openai|OpenAI]]'s first tool platform for [[chatgpt|ChatGPT]]. It lasted about a year: **2023-03-23 → 2024-04-09**.

## Timeline

| Date | Event | Source |
|---|---|---|
| **2023-03-23** | Alpha launch: third-party plugins, plus OpenAI's own browsing and code interpreter plugins and an open-source retrieval plugin | [[chatgpt-plugins-openai-2023]] |
| 2023-06-13 | [[function-calling\|Function calling]] in the API: tool schemas move into each request, with models fine-tuned to emit JSON arguments. The post cites "the alpha release of ChatGPT plugins" and a tool-output injection exploit. | [[function-calling-api-updates-openai-2023]] |
| 2023-06-23 | Weng names plugins and function calling as the two working examples of tool use | [[llm-powered-autonomous-agents-weng-2023]] |
| 2023-11-06 | **GPTs and Actions**: "the design of actions builds upon insights from our plugins beta"; existing plugin manifests can define actions | [[introducing-gpts-openai-2023]] |
| **2024-03-19** | No new plugin installs or conversations | [[chatgpt-plugins-wind-down-2024]] (secondhand) |
| **2024-04-09** | Existing plugin conversations end; the store had a little over 1,000 plugins | same |

## How it worked

- **Developer side:**
  1. Build or wrap an API.
  2. Document it with an **OpenAPI** spec.
  3. Write a **manifest**: model-facing name and description, API URL, auth.
- **Run time:** the user enables up to a few plugins; their documentation is **inserted into the prompt**; the model decides when to call.
- **Built-in plugins:**
  - **Browsing:** a descendant of [[webgpt|WebGPT]]; GET requests only, via the Bing API; cites its sources; honours robots.txt.
  - **Code interpreter:** sandboxed, firewalled Python with a persistent session.
- **Retrieval plugin:** open-source, self-hosted vector search over one's own documents.
- **Safety:** red-teamers found that plugins without safeguards enabled "sophisticated [[prompt-injection|prompt injection]]", spam and fraud. That class of risk was carried into the function-calling post as **indirect prompt injection** through tool output.

## Why it matters

1. **It moved tool use from research into a product:** [[toolformer|Toolformer]] and [[react|ReAct]] (2022–23) → plugins.
2. **It showed the cost of putting tool descriptions in the prompt:** only a few plugins could be enabled at once. Today's progressive disclosure in [[agent-skills]] and [[context-engineering]] addresses the same problem.
3. **Its "AI-facing interface" standard** was predicted in the launch post and arrived as [[mcp|MCP]] from Anthropic (2024-11-25).
4. **The name lives on with a different meaning:** Claude Code *plugins* (2025-10-09) are bundles of skills, hooks, commands and MCP servers for a coding agent, not tool APIs for a chat app ([[claude-code-changelog-2026]]).

## References

- [[chatgpt-plugins-openai-2023]]
- [[function-calling-api-updates-openai-2023]]
- [[introducing-gpts-openai-2023]]
- [[chatgpt-plugins-wind-down-2024]]
- [[llm-powered-autonomous-agents-weng-2023]]
