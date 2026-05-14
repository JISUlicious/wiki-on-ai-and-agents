---
title: "opencode — Architecture & Implementation Notes"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - opencode.md
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
  - "[[opencode]]"
---

# opencode — Architecture & Implementation Notes

**Source**: `sources/opencode.md` (57KB technical document)
**Origin**: Architectural writeup of `sst/opencode` (May 2026).

## Summary

[[opencode]] (npm package `opencode-ai`, binary `opencode`) is an open-source, provider-agnostic coding agent by the team behind SST and terminal.shop. Tagline: "the open source AI coding agent." Among the most-starred open-source alternatives to [[claude-code|Claude Code]].

Architecturally: a TypeScript/Bun monorepo built around a **headless HTTP/SSE server** that hosts the agent loop, with multiple front-ends:

- Terminal-native TUI (SolidJS + OpenTUI)
- CLI `run` mode
- Electron desktop app
- Agent Client Protocol (ACP) bridge for Zed
- OpenCode web/console (hosted)

Built on Effect runtime (effect/Service, Layer, Schema). Uses the Vercel AI SDK (`ai`, `@ai-sdk/*`) for model calls. Drizzle/SQLite (Bun's native driver) for durable storage of sessions, messages, parts, and an event log.

## Distinguishing features

- **Strict client/server split**: the same agent can be driven from terminal, desktop, mobile, or hosted console.
- **Model catalog ingested live from `https://models.dev`** — 23 bundled providers, dynamically updatable.
- **Event-sourcing layer**: sync-event log behind sessions enables replay + the famous "shareable sessions" feature.
- **Plugin / skill / agent extension surface mirrors Claude Code conventions** (`.claude/skills`, `AGENTS.md`, custom subagents, MCP).

## Key architectural facts

- **Language/runtime**: 100% TypeScript on Bun.
- **Server-first**: agent loop lives in a headless HTTP/SSE server; UIs are clients.
- **Tools**: ~17 built-in (shell, read, edit, write, glob, grep, task, fetch, search, …).
- **MCP**: first-class — stdio + HTTP + SSE + OAuth.
- **ACP**: yes — `opencode acp` for Zed integration.
- **Sub-agents**: first-class via `task` tool; built-in `general`/`explore`/`plan`/`compaction` agents.
- **Sandbox**: none in core; experimental "containers" workspace adapter.
- **Permissions**: pattern-rule engine, persisted per project, `Permission.ask/reply` over SSE.
- **Memory**: session summaries; snapshot side-git for revert; durable sessions.
- **Skills format**: SKILL.md, walks `~/.claude/skills/`, `.agents/skills/`, and project dirs. Compatible with Claude Code's conventions.

## Entities Mentioned

- [[opencode]] — the agent
- [[claude-code]] — major reference point and compatibility target

## Concepts Discussed

- [[llm-agent]] — opencode is a server-architected instance
- [[tool-use]]
- Provider abstraction via Vercel AI SDK + models.dev
- Event sourcing for replayable sessions
- Skills format interoperability

## References

_Original source: `sources/opencode.md`_
_Also see [[cross-agent-comparison-2026|the cross-agent comparison]]._
