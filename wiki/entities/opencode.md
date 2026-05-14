---
title: opencode
type: entity
created: 2026-05-14
updated: 2026-05-14
sources:
  - opencode.md
  - comparison.md
family: agent-product
parameters: "n/a (provider-agnostic)"
release_date: 2024-09-01
status: complete
importance: medium
tags:
  - product
---

# opencode

opencode (`sst/opencode`, npm package `opencode-ai`, binary `opencode`) is an open-source, provider-agnostic coding agent built by the SST + terminal.shop team. Tagline: "the open source AI coding agent."

## Architecture

- **TypeScript / Bun monorepo** built around a **headless HTTP/SSE server** that hosts the agent loop.
- **Multiple front-ends**: terminal-native TUI (SolidJS + OpenTUI), CLI `run` mode, Electron desktop, ACP bridge for Zed, hosted web/console.
- **Effect runtime** (effect/Service, Layer, Schema) throughout the core.
- **Vercel AI SDK** (`ai`, `@ai-sdk/*`) for model calls.
- **Drizzle/SQLite** (Bun native driver) for durable storage of sessions, messages, parts, event log.
- **Live model catalog** from `https://models.dev` — 23 bundled providers, dynamically refreshable.
- **Sync-event log** behind sessions enables replay and "shareable sessions."

## Distinguishing features vs. other 2026 agents

- **Strict client/server split**: same agent driven from terminal, desktop, mobile, hosted console. Among the documented 2026 agents, the most explicit about being a deployable service rather than a single-binary CLI.
- **Model catalog as live config**: pulls from models.dev rather than baking in a provider list.
- **Skills/agent extension surface mirrors Claude Code conventions** (`.claude/skills`, `AGENTS.md`, custom subagents, MCP). High interop with the broader Claude Code ecosystem.
- **One of the most-starred open-source alternatives to [[claude-code|Claude Code]]**.

## Tool inventory and capabilities

- **~17 built-in tools**: shell, read, edit, write, glob, grep, task, fetch, search, ...
- **MCP**: first-class — stdio + HTTP + SSE + OAuth.
- **ACP**: `opencode acp` for Zed integration.
- **Sub-agents**: first-class via `task` tool. Built-in `general`, `explore`, `plan`, `compaction` agents.
- **Sandbox**: none in core; experimental "containers" workspace adapter.
- **Permissions**: pattern-rule engine, persisted per project, `Permission.ask/reply` over SSE.
- **Memory**: session summaries; snapshot side-git for revert; durable sessions.
- **Skills format**: SKILL.md; walks `~/.claude/skills/`, `.agents/skills/`, and project dirs.

## References

- [[opencode-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]
