---
title: "opencode codebase study (anomalyco/opencode @ 79bd2c6f11, November 2026)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - opencode-codebase-study-2026-11-18.md
year: 2026
url: https://github.com/anomalyco/opencode
status: complete
importance: medium
tags:
  - 2026
---

# opencode codebase study (November 2026)

Read on 2026-11-18 from [anomalyco/opencode](https://github.com/anomalyco/opencode) at commit `79bd2c6f11` (2026-11-18, all packages at version 2.3.0, "OpenCode V2"). Only source files and docs were read. Nothing was installed, built or run, so this page records what the code and docs say, not how well it works. Raw notes with file paths: `sources/opencode-codebase-study-2026-11-18.md`.

## Summary

[[opencode]] has been rewritten as **V2**. The agent now lives in `packages/core` (Effect services), model calls go through opencode's own `@opencode/ai` package rather than the Vercel AI SDK, and almost everything (tools, agents, Plan mode, `/goal`, per-model prompts, input repair) is a built-in plugin. The biggest new idea is **Code Mode**: one `execute` tool runs model-written JavaScript in opencode's own interpreter, and MCP tools and utility tools sit behind it by default. Sub-agents now always run in the background. The V1 hook names and permission format are gone.

## Architecture

- **Language/runtime**: TypeScript on Bun (`bun@1.4.4`), Effect throughout Core. SQLite through Drizzle, with sessions stored as an event log and projected into tables.
- **Process model**: a background service hosts Core (`opencode service`, `serve`). TUI (OpenTUI + Solid), Electron desktop, web app, `run`, `mini` and [[acp|ACP]] are clients of it.
- **Packages**: `core` (agent, ~225k lines), `ai` (providers, ~95k), `codemode`, `plugin`, `server`, `client`, `cli`, `tui`, `app`, `desktop`, plus smaller ones. The old `packages/opencode` is gone.
- **Providers**: `@opencode/ai` has ~60 wire protocols and ~50 provider modules. An AI SDK adapter still loads third-party provider packages. A models.dev snapshot is bundled.

## Agent loop

`SessionRunner.drain` in `packages/core/src/session/runner/llm.ts` runs one turn. It reads the durable session **inbox**, runs queued control items (manual compaction, session move), promotes pending input, then calls `runStep` until a step ends without tool calls.

- **Steering**: input arrives as `steer` (promoted between steps of the current turn) or `queue` (waits for the next turn).
- **Stop condition**: a step whose response has no local tool calls. Before going idle, the runner fires `experimental.idle`; if a plugin admits input there, the turn continues. This is how `/goal` works.
- **Parallel tools**: each `tool-call` event is forked as soon as it streams in, so tools run concurrently and start before the response finishes. File writes are serialized per path.
- **Retries**: exponential backoff from 2 s, capped at 10 s, up to 10 retries. `retry-after` is honored up to 15 min, timeouts get 3 retries, and network-unreachable errors get a separate 1-hour budget. The `retry` hook can change the decision.
- **Limits and guards**: an optional per-agent `steps` cap (unset by default) forces a text-only final answer. Three `length` finishes in a row fail the step. A provider overflow triggers one compaction and a retry. Server restarts resume work up to 10 times. No repeated-call ("doom loop") detector was found.

## Context & memory

- **System prompt**: a 15-line generic base prompt with per-tool guidance. Per-model plugins override it (GPT, Kimi, Arcee, Meta) or append to it (Claude, two lines on comment style).
- **Instructions** ([[context-assembly]]): global `AGENTS.md`, then each `AGENTS.md` from the working directory up to the project root. `CLAUDE.md` is used where no `AGENTS.md` exists. After that come configured instructions, then date and environment. Nested `AGENTS.md` files load lazily when `read` touches their area. Edits are sent as **instruction update** messages instead of rewriting the prompt, which keeps the prompt cache warm.
- **[[context-compaction]]**: runs automatically when the conversation fills all but a buffer (15% of the window, at least 16k tokens), once on provider overflow, or on request. It summarizes the whole conversation (to reuse the prompt cache) into a fixed template (Objective, Requirements, Decisions, Work State, Next Move, Relevant Files, Important Context) and keeps the last ~15k tokens verbatim. Summaries are precomputed in the background. Models with native compaction use the provider's own.
- **Memory**: no long-term store. Memory is `AGENTS.md`, the SQLite session log, compaction summaries and a side-git snapshot used for revert.

## Tools & MCP & sub-agents

- **Direct tools** (13 by default): `read`, `glob`, `grep`, `edit`, `write` (`patch` instead of these two for GPT models), `shell`, `webfetch`, `websearch`, `question`, `skill`, `subagent`, and `execute`. `experimental_image_generate` and `todowrite` are conditional, and `todowrite` is off in release builds.
- **Code Mode** ([[programmatic-tool-calling]], [[code-as-harness]]): `execute` runs a JavaScript-like language in opencode's own interpreter. It has no filesystem, process or timers, only `fetch` and catalog tools. The catalog has 8 `opencode` utilities (session rename/move, worktrees, models, MCP resources), flag-gated artifacts and automations, browser tools and MCP tools. Nested calls still pass permission checks.
- **[[mcp|MCP]]**: stdio and streamable HTTP with OAuth. No SSE transport was found. Experimental MCP events are behind a flag. MCP tools go into Code Mode unless a server sets `codemode: false`.
- **[[subagents|Sub-agents]]**: the `subagent` tool (V1 `task`) starts a child session in the background and returns its ID. The parent gets a `<subagent>` message each time the child goes idle. Default nesting depth is 1. Built-in agents: Build, Plan, General, Explore, plus hidden Compaction, Title and Summary.
- **Skills** ([[agent-skills]], [[skill-md-format]]): `SKILL.md` from `.opencode/skills`, `.claude/skills` and `.agents/skills` (global and project), plus HTTP catalogs. They are listed in instructions and loaded by the `skill` tool.

## Hooks

21 named hooks in 5 domains (`packages/plugin/src/effect/*`). Handlers mutate the event in place, in registration order. Only `tool.execute.before` can fail, which blocks the call.

| Hook | Fires | Block / modify? |
|---|---|---|
| `tool.execute.before` | before every tool call, including inside Code Mode | rewrite name/input; **block** with `Tool.Error` |
| `tool.execute.after` | after a tool call | rewrite result or error |
| `session.prompt` | user input admitted | edit prompt, delivery |
| `session.context` | before each main-loop request | edit system, messages, tools, options |
| `session.compaction` / `generate` / `title` | auxiliary requests | edit; supply a result and skip the model |
| `session.experimental.model.resolve` | model selection per request | swap model |
| `session.model.request`, `http.request`, `http.response`, `experimental.ws.*` (3) | transport | edit URL, headers, HTTP, WebSocket frames |
| `session.retry` | retry decision | veto or change delay |
| `session.experimental.idle` | before the session goes idle | admit input to keep going (Stop-hook style) |
| `session.fork` | session forked | observe |
| `permission.evaluate` | each permission check | change allow/ask/deny |
| `shell.create.before` | before a shell spawns | edit command, cwd, env, timeout |
| `aisdk.sdk`, `aisdk.language` | provider construction | supply SDK or model instance |

## Guardrails

| Layer | What exists | Where |
|---|---|---|
| Permission rules | ordered `{action, resource, effect}`; last match wins; no match means `ask` | `core/src/permission.ts`, docs `permissions.mdx` |
| Defaults | Build agent allows everything except `.env` reads (ask); `external_directory` allowed | `schema/src/agent.ts` |
| Saved approvals | per project | `permission/saved.ts` |
| Unattended runs | anything that would ask is denied | `Permission.Unattended` |
| Agent scoping | Plan denies edits outside its plan dir; Explore is read-only | `plugin/plan.ts`, `plugin/agent.ts` |
| Sandbox | **none** for shell; Code Mode interpreter is the only confined runtime | `shell.ts`, `packages/codemode` |
| Limits | output 2000 lines / 50 KiB, retry budgets, optional step cap | `tool-output.ts`, `model-retry.ts` |

**Goal command**: `/goal` is a built-in plugin with up to 50 rounds. A reviewer request with no tools returns `done`, `continue`, `blocked`, `paused` or `wait`. Three stalls (the same reason twice, or no tool use) pause the goal. There is no other built-in self-check step ([[verifier]]).

## Component map (agent core)

| Component | Path (`packages/core/src`) | ~LOC | Plugs in |
|---|---|---|---|
| Loop / turn runner | `session/runner/`, `session/execution.ts`, `session/inbox.ts` | 3.3k | is the loop |
| Tools | `tool.ts`, `tool/plugin/*` | 6.3k | snapshot per request, forked per call |
| Code Mode | `codemode/` + `packages/codemode` | 1.6k + 22k | single `execute` tool |
| Context assembly | `session/model-request.ts`, `instruction-state.ts`, `instructions/` | 2.1k | each step + `context` hook |
| Compaction | `session/compaction.ts` | 1.3k | before each call, on overflow |
| Goal / plan / todo | `plugin/goal.ts`, `plugin/plan.ts` | 0.75k | `idle` + `context` hooks |
| Sub-agents | `tool/plugin/subagent.ts`, `session/subagent-*` | 0.9k | child sessions |
| Skills | `skill.ts`, `skill/` | 0.65k | instructions + tool |
| Hooks | `plugin/hooks.ts` | 1.6k | 21 trigger sites |
| Permissions | `permission.ts` | 0.75k | each tool leaf |
| MCP client | `mcp/` | 4.0k | Code Mode catalog |
| Recovery | `plugin/tool-input-repair.ts`, `execution/restart.ts` | 0.9k | wraps tools and steps |

## Changes since the earlier wiki pages

- **Version and layout**: V2 (2.3.0). `packages/core` replaces `packages/opencode`.
- **Providers**: own `@opencode/ai`, not the Vercel AI SDK. "23 bundled providers" is out of date.
- **Hooks**: 21 V2 hooks, not "~18". `chat.params`, `chat.message`, `permission.ask` and `experimental.session.compacting` are gone.
- **Tools**: 13 direct tools plus a Code Mode catalog, not "~17". `bash` is now `shell` and `task` is now `subagent`. The `doom_loop` and `lsp` permissions are gone.
- **MCP**: no SSE transport found. MCP tools sit in Code Mode by default.
- **Sub-agents**: always background (v2.3.0), depth 1.
- **Goal**: still 50 rounds with a reviewer that uses no tools. New: stall detection and a `wait` verdict.

## Concepts & entities

[[opencode]], [[claude-code]], [[pi-mono]], [[cline]], [[codex-cli]], [[zed]], [[mcp]], [[acp]], [[programmatic-tool-calling]], [[code-as-harness]], [[context-compaction]], [[context-assembly]], [[context-engineering]], [[subagents]], [[agent-skills]], [[skill-md-format]], [[verifier]], [[ralph-loop]], [[agentic-harness-engineering]].

## References

- Repo: https://github.com/anomalyco/opencode @ `79bd2c6f11`
- Earlier pages: [[opencode-agent-doc-2026]], [[cross-agent-comparison-doc-2026]], [[agent-capabilities-check-2026-11]], [[goal-command-lineage-repos-2026]], [[agent-products-repo-history-2026]]
- Wiki syntheses: [[cross-agent-comparison-2026]], [[hooks-in-modern-agents]], [[fundamental-components-of-harness]], [[agent-harness-code-study-2026-11]]
