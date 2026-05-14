---
title: Hermes Agent
type: entity
created: 2026-05-14
updated: 2026-05-14
sources:
  - hermes-agent.md
  - comparison.md
family: agent-product
parameters: "n/a (provider-agnostic, OpenAI-format chat completions)"
release_date: 2025
status: complete
importance: medium
tags:
  - product
---

# Hermes Agent

Hermes Agent (`NousResearch/hermes-agent`) is **Nous Research**'s open-source, self-improving AI agent. Python-based. Runs as either an interactive terminal (`hermes`) or a persistent multi-platform messaging daemon (`hermes gateway`). Tagline from its README:

> "The self-improving AI agent built by Nous Research. It's the only agent with a built-in learning loop — it creates skills from experience, improves them during use, nudges itself to persist knowledge, searches its own past conversations, and builds a deepening model of who you are across sessions."

Provider-agnostic via OpenAI-compatible chat completions. Works with: OpenRouter, Nous Portal, [[anthropic|Anthropic]], NVIDIA NIM, Hugging Face, Ollama, vLLM, llama.cpp, LM Studio.

## Distinguishing feature: self-modifying runtime

What sets Hermes apart from the other documented 2026 agents: **the runtime *and* the persistence layer (skills, memory, sessions, cron jobs) are themselves edited by the agent during normal use**. "Growing" is treated as a first-class system concern, not a UX flourish.

Concretely:
- The agent can create new skills from experience and modify them during use.
- Background "nudging" mechanism for persisting knowledge.
- 4-layer memory: frozen MEMORY.md/USER.md snapshot + provider plugin + FTS5 session search + auto-archive curator.

## Codebase scale

- ~17k tests across ~900 files (per the development guide, May 2026).
- `run_agent.py` — core conversation loop, ~15.7k LOC.
- `cli.py` — CLI orchestrator, ~13.5k LOC.
- `hermes_state.py` — SessionDB on SQLite + FTS5 search.

## Architecture

- **Multi-surface**: terminal, chat platforms (Telegram, Discord, etc. via gateway), ACP adapter for VS Code / Zed / JetBrains.
- **Sandboxing**: 7 backends — local, Docker, SSH, Singularity, Modal, Daytona, Vercel Sandbox.
- **Sub-agents**: `delegate_task` with leaf/orchestrator roles, capped depth.
- **Permissions**: per-tool approval callback + ACP `request_permission`.
- **MCP**: first-class (stdio / HTTP / SSE).
- **Skills**: SKILL.md (Anthropic-inspired + agentskills.io compatible); bundled + optional skills.
- **Tools**: 25+ built-in including memory, session_search, terminal, delegate, kanban.

## References

- [[hermes-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]
