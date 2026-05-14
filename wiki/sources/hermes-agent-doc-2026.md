---
title: "Hermes Agent — Architecture & Implementation Notes"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - hermes-agent.md
arxiv_id: null
venue: Technical write-up (latest-agents research corpus)
authors:
  - "(unattributed; internal technical doc)"
year: 2026
tags:
  - 2026
status: complete
importance: medium
introduces:
  - "[[hermes-agent]]"
---

# Hermes Agent — Architecture & Implementation Notes

**Source**: `sources/hermes-agent.md` (50KB technical document)
**Origin**: Architectural writeup of `NousResearch/hermes-agent`, Nous Research's open-source self-improving agent (as of May 2026).

## Summary

[[hermes-agent|Hermes Agent]] is a Python-based open-source AI agent from **Nous Research**. The distinguishing claim from its README:

> "The self-improving AI agent built by Nous Research. It's the only agent with a built-in learning loop — it creates skills from experience, improves them during use, nudges itself to persist knowledge, searches its own past conversations, and builds a deepening model of who you are across sessions."

Architecturally:

- Two entry surfaces: interactive terminal (`hermes`) and persistent messaging daemon (`hermes gateway`)
- Provider-agnostic via OpenAI-compatible chat completions: works with OpenRouter, Nous Portal, Anthropic, NVIDIA NIM, Hugging Face, Ollama, vLLM, llama.cpp, LM Studio
- ~17k tests across ~900 files
- Heavily plugin-oriented; runtime + persistence layer (skills, memory, sessions, cron jobs) are themselves editable by the agent during use

Top-level layout (verified line counts in the source doc):
- `run_agent.py` — `AIAgent` class, core conversation loop (~15.7k LOC)
- `cli.py` — `HermesCLI` interactive orchestrator (~13.5k LOC)
- `hermes_state.py` — SessionDB on SQLite with FTS5 search
- `agent/`, `hermes_cli/`, `tools/`, `gateway/`, `plugins/`, `skills/`, `ui-tui/`, `acp_adapter/`

The defining design choice: **"growing" is treated as a first-class system concern, not a UX flourish**. The agent edits its own skills and memory layer during normal use. This is unique among current production agents.

## Key architectural facts

- **Multi-surface**: terminal, chat platforms (Telegram, Discord, etc. via gateway), ACP (VS Code / Zed / JetBrains).
- **Memory: 4 layers** — frozen MEMORY.md/USER.md snapshot + provider plugin + FTS5 session search + auto-archive curator.
- **Sub-agents**: `delegate_task` with leaf/orchestrator roles, capped depth.
- **Sandboxing**: 7 terminal backends — local, Docker, SSH, Singularity, Modal, Daytona, Vercel Sandbox.
- **Skills**: SKILL.md format (Anthropic-inspired + agentskills.io compatible); bundled + optional skills.
- **Self-improving loop**: the agent can create new skills from experience and modify them during use, with a background "nudging" mechanism for persisting knowledge.

## Entities Mentioned

- [[hermes-agent]] — the agent
- [[nous-research]] — origin org

## Concepts Discussed

- [[llm-agent]] — Hermes Agent is a notable multi-surface instance
- Self-modifying agent architecture
- [[tool-use]]
- Memory layering for long-running agents

## References

_Original source: `sources/hermes-agent.md`_
_Also see [[cross-agent-comparison-2026|the cross-agent comparison]]._
