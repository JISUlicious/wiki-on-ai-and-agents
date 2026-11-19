---
title: opencode
type: entity
created: 2026-05-14
updated: 2026-11-18
sources:
  - opencode.md
  - comparison.md
  - opencode-codebase-study-2026-11-18.md
family: agent-product
parameters: "n/a (provider-agnostic)"
release_date: 2024-09-01
status: complete
importance: medium
tags:
  - product
---

# opencode

opencode (`anomalyco/opencode`, formerly `sst/opencode`; npm package `opencode-ai`, binary `opencode`) is an open-source, provider-agnostic coding agent built by the SST + terminal.shop team. Tagline: "the open source AI coding agent."

## Architecture

As of commit `79bd2c6f11` (2026-11-18, version 2.3.0, "OpenCode V2"), from [[opencode-codebase-study-2026]]:

- **TypeScript / Bun monorepo**. The agent runs as Effect services in `packages/core`, inside a **background service** (`opencode service`, `serve`).
- **Multiple front-ends**: TUI (OpenTUI + Solid), `run` and `mini` CLI modes, Electron desktop, web app, and an ACP bridge (`opencode acp`).
- **Own provider layer**: `@opencode/ai` (~60 wire protocols, ~50 provider modules) plus an adapter that still loads AI SDK provider packages. A models.dev snapshot is bundled.
- **Drizzle/SQLite** event log for sessions, projected into tables. A side-git snapshot supports revert.
- **Agent loop**: `SessionRunner.drain` reads a durable inbox (`steer` or `queue` delivery) and runs steps until a response has no tool calls. Tools are forked as each call streams in, so they run in parallel. An `experimental.idle` hook can keep the turn going.

> [!warning] Contradiction (2026-11-18)
> Earlier text said opencode uses the **Vercel AI SDK** for model calls and has **23 bundled providers**. The V2 code calls models through its own `@opencode/ai` package, and the AI SDK survives only as an adapter ([[opencode-codebase-study-2026]]).

## Distinguishing features vs. other 2026 agents

- **Strict client/server split**: same agent driven from terminal, desktop, mobile, hosted console. Among the documented 2026 agents, the most explicit about being a deployable service rather than a single-binary CLI.
- **Model catalog as live config**: pulls from models.dev rather than baking in a provider list.
- **Skills/agent extension surface mirrors Claude Code conventions** (`.claude/skills`, `AGENTS.md`, custom subagents, MCP). High interop with the broader Claude Code ecosystem.
- **Everything is a plugin (V2)**: built-in tools, agents, Plan mode, `/goal`, per-model prompts and tool-input repair all register through the same plugin API that third-party plugins use.
- **Code Mode by default**: one `execute` tool fronts a catalog of utility and MCP tools, so their schemas are not all sent as direct tools.
- **One of the most-starred open-source alternatives to [[claude-code|Claude Code]]**.

## Tool inventory and capabilities

- **Tools**: 13 direct tools by default: `read`, `glob`, `grep`, `edit`, `write` (`patch` instead of these two for GPT models), `shell`, `webfetch`, `websearch`, `question`, `skill`, `subagent`, and `execute` (Code Mode). `experimental_image_generate` and `todowrite` are conditional.
- **Code Mode**: `execute` runs model-written JavaScript in opencode's own interpreter, which has no filesystem, process or timers. Utility tools, browser tools and MCP tools sit behind it by default ([[programmatic-tool-calling]]).
- **MCP**: stdio and streamable HTTP with OAuth. Experimental MCP events are behind a flag.
- **ACP**: `opencode acp` (Zed and other ACP clients).
- **Sub-agents**: `subagent` tool. Children always run in the background (v2.3.0) and report back when idle. Default depth is 1. Built-in agents: Build, Plan, General, Explore, plus hidden Compaction, Title and Summary.
- **Sandbox**: none for shell commands. Code Mode's interpreter is the only confined runtime.
- **Permissions**: ordered `permissions` rules `{action, resource, effect}`. The last match wins, and no match means `ask`. The default agent allows everything except `.env` reads. Approvals can be saved per project.
- **Hooks**: 21 named V2 hooks in 5 domains (tool, session, permission, shell, aisdk). Only `tool.execute.before` can block.
- **Memory**: AGENTS.md (with a CLAUDE.md fallback), durable SQLite sessions and compaction summaries. There is no long-term memory store.
- **Compaction**: automatic near the context limit (15% buffer, at least 16k tokens). It summarizes into a fixed template, keeps ~15k recent tokens, and precomputes in the background.
- **Skills format**: SKILL.md from `.opencode/skills`, `.claude/skills` and `.agents/skills`, globally and in the project.

> [!warning] Contradiction (2026-11-18)
> Earlier text said "~17 built-in tools" (shell, task, fetch, ...), MCP over "stdio + HTTP + SSE", built-in agents "general, explore, plan, compaction", and an "experimental containers workspace adapter". The V2 code has 13 direct tools plus a Code Mode catalog. `bash` is now `shell` and `task` is now `subagent`. No SSE MCP transport was found. Title and Summary are also built-in agents. The workspace layer is an abstract `WorkspaceDriver`, not a container sandbox. The `doom_loop` permission is gone. Older wiki syntheses cite ~18 plugin events (`chat.params`, `permission.ask`, `experimental.session.compacting`); those V1 names are not in V2 ([[opencode-codebase-study-2026]]).

**Goals and schedules:** `/goal` (2026-10-29) is a built-in plugin. A reviewer step that calls no tools judges each round, with a cap of 50 rounds. As of 2026-11-18 it also pauses after 3 stalled rounds and can `wait` for a running shell. Automations (scheduled tasks) are behind the `automations` flag. See [[goal-command-lineage-repos-2026]].

## References

- [[opencode-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]
- [[opencode-codebase-study-2026]]

## Related 2026 sources

- [[illicit-distillation]] — named in Anthropic's 2026 report: DeepSeek allegedly tagged OpenCode users by request strings and relayed them to Claude Opus, and Xiaomi replayed OpenCode sessions through Claude for training data

## Lineage sources

- [[harness2-recursive-agent-harnessing-xu-2026]] — the default executor substrate and editor runtime in Harness²
