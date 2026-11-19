---
title: goose
type: entity
created: 2026-11-18
updated: 2026-11-18
sources:
  - goose-codebase-study-2026-11-18.md
family: agent-product
parameters: "n/a (provider-agnostic; 35 native + 56 declarative provider definitions)"
status: complete
importance: medium
tags:
  - product
---

# goose

goose is a general-purpose, open-source AI agent ("not just for code"). It is written in Rust under the Apache-2.0 license and ships as a desktop app, a CLI and an embeddable server. [[block|Block]] created it at `block/goose`. It now lives at `aaif-goose/goose`. `GOVERNANCE.md` says the project is "now stewarded by AAIF" (the Agentic AI Foundation at the Linux Foundation) and is set up as a Series of LF Projects, LLC. The system prompt now says the agent was "created by AAIF". The shallow clone does not show when the repo moved.

Code-level detail is in [[goose-codebase-study-2026]] (commit `927005e143`, v1.60.0, read 2026-11-18).

## At a glance

- **Language and runtime**: a Rust workspace of 15 crates; the core `goose` crate is about 195k lines. The Electron desktop app is in `ui/desktop`. A publishable subset, the "GDK", includes the loop crate `goose-agent`. It has Python/Kotlin bindings and also builds for wasm.
- **Surfaces**:
  - the `goose` CLI (`session`, `run`, `recipe`, `schedule`, `term` and more);
  - the desktop app, which spawns `goose serve` and speaks [[acp|ACP]] over WebSocket (the old `goosed` server is gone);
  - `goose acp` for IDEs;
  - `goose roam` for peer-to-peer access over iroh;
  - a Telegram gateway.
- **Providers**: many API providers plus local [[llama-cpp|llama.cpp]]. Through ACP providers it can also drive [[claude-code|Claude Code]], [[codex-cli|Codex]], Copilot, Amp, [[pi-mono|pi]] and [[cursor|Cursor]] as if they were models.
- **Loop**: an ordered state machine of about 25 operations. On each step the first operation that applies wins, and model inference comes last. Tool calls run in parallel. The turn cap defaults to 1000. `/goal` nudges the model once to check the goal; `/grind` keeps nudging until the turn cap.
- **Tools**: all tools are MCP "extensions":
  - 11 in-process platform extensions with 25 tools, 19 of them on by default;
  - 4 bundled MCP servers with 17 tools;
  - any external [[mcp|MCP]] server.
  Sub-agents come from `summon.delegate`: one level deep, foreground only, up to 5 in parallel ([[subagents]]). [[agent-skills|Skills]] are SKILL.md files loaded with `load_skill`. Recipes are YAML workflows with sub-recipes, retry checks and cron scheduling.
- **Context**: `.goosehints` and `AGENTS.md` hints, plus a per-turn `<turn-context>` block. [[context-compaction|Compaction]] runs at 0.8 × the context limit and writes an LLM summary; old messages stay visible to the user but are hidden from the agent.
- **Hooks**: 12 events in the Open Plugins hook format, loaded from plugins. Only `PreToolUse` and `Stop` can block; see [[hooks-in-modern-agents]].
- **Guardrails**: the modes are `auto` (the default), `approve` and `smart_approve`. Smart approve sends unknown tools to an [[llm-as-a-judge|LLM judge]] that decides whether they are read-only. Several layers are opt-in: an injection/command scanner (off by default), an "adversary" LLM reviewer, and an OSV malware check on extensions. There is **no OS sandbox**; the only isolation is an opt-in Docker `--container`. See [[guardrailing]].

## Lineage

goose began as Block's internal and then open-source agent, which explains the leftover "Block" naming in config paths. It moved under AAIF's neutral foundation governance. Recent architecture changes include:

- replacing the `goosed` REST server with an ACP server;
- retiring the old `chat` mode (it now loads as `approve`);
- removing background sub-agents;
- moving the MCP server directory to the official MCP Registry.

## References

- [[goose-codebase-study-2026]]
- https://github.com/aaif-goose/goose
