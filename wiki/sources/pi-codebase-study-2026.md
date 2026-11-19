---
title: "Pi codebase study (earendil-works/pi @ f6d0c2f9b6, November 2026)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - pi-codebase-study-2026-11-18.md
year: 2026
url: https://github.com/earendil-works/pi
status: complete
importance: medium
tags:
  - 2026
---

# Pi codebase study (November 2026)

Read on 2026-11-18 from [earendil-works/pi](https://github.com/earendil-works/pi) at commit `f6d0c2f9b6` (2026-11-18, package version 1.5.2). Only source files and docs were read. Nothing was installed, built or run, so this page records what the code and docs say, not how well it works. Raw notes with file paths: `sources/pi-codebase-study-2026-11-18.md`.

## Summary

[[pi-mono|Pi]] is still a small-core coding agent: 4 tools are active by default and there is still no built-in permission system, sandbox for shell commands, sub-agent tool, plan mode or goal command. The surface around that core has grown a lot since the May 2026 doc. Pi now ships built-in MCP, a **codemode** tool that runs model-written JavaScript in a QuickJS/WASM VM, deferred **tool search**, a **project-trust** gate, virtual (routing) models, 43 provider modules, and 41 extension events. The monorepo has also gained experimental durable and server packages (`pi-durable`, `pi-env`, `chord`, `apps/p2`, `apps/p3`).

## Architecture

- **Language/runtime**: TypeScript, ESM, Node ≥22.19 (`packages/coding-agent/package.json` `engines`). Bun is only a compile target for standalone binaries.
- **Packages**: `pi-ai` (providers), `pi-agent-core` (loop), `pi-coding-agent` (the `pi` CLI, SDK, RPC entry), `pi-tui`, plus new `pi-mcp`, `pi-codemode`, `pi-schema`, `pi-telemetry`, `pi-durable`, `pi-env`, `chord`, `pi-evals`. The CLI depends only on agent-core, ai, codemode, mcp and tui. `pi-web-ui` is gone from `packages/`.
- **Surfaces**: interactive TUI, print, JSON (event JSONL), and RPC (JSONL over stdin/stdout) modes, plus the in-process TypeScript SDK (`src/core/project-trust.ts` `AppMode`; `docs/how-pi-works.md`). Web and desktop are experimental apps (`apps/p2` web UI "will be gone soon", `apps/p2-mac` menu-bar spike, `apps/p3` headless per-user server). No ACP was found.
- **Process model**: one Node process. Codemode scripts run in worker threads (max 2), MCP stdio servers are child processes, and the example sub-agent extension spawns separate `pi` processes.
- **Providers**: `pi-ai` has 10 chat wire APIs (`KnownApi` in `packages/ai/src/types.ts`), 1 image API and 4 classifier APIs, with 43 `*.models.ts` provider files. It includes a first-party gateway, **Radius** (`radius.pi.dev`).

## Agent loop

`runLoop()` in `packages/agent/src/agent-loop.ts` has an outer follow-up loop and an inner loop that runs while there are tool calls or steering messages. Each turn goes: `prepareNextTurn` (compaction happens here) → steering poll → `turn_start` → declare tool changes → `prepareRequest` → stream the response → run tools → `finishTurn` decision → steering poll. When the inner loop ends, follow-up messages restart it. Otherwise a `continue` decision buys one more context-only turn, or the run ends.

- **Tools run in parallel by default** (`Agent.toolExecution = "parallel"`). A tool marked `executionMode: "sequential"` makes the whole batch sequential. Edits are serialized per file by `withFileMutationQueue`.
- If `stopReason === "length"`, every tool call in that message is failed rather than run with truncated arguments.
- A batch whose results all set `terminate: true` ends the run.
- Tool loadout changes are written as `toolsAdded`/`toolsRemoved` on a system message appended to the transcript. The request head is not rewritten, which keeps the prompt cache valid.
- Queues default to `one-at-a-time` for both steering and follow-up. Retry defaults are 3 attempts with a 2000 ms base delay. A context overflow or `length` stop gets one compact-and-retry attempt.
- Settle lifecycle: `agent_end` → `agent_before_settle` (can force one more request) → `agent_settled`.
- Nested tool calls from codemode go through the same hooks, carry `parentToolCallId`, and never enter the LLM context (`src/core/nested-tool-calls.ts`).

## Context & memory

- **System prompt** is built in sections (`src/core/system-prompt.ts`): preamble/`SYSTEM.md`, tools, rules, a pointer to Pi's own docs, `APPEND_SYSTEM.md`, project context, skills, cwd, and MCP servers. When a section changes mid-session, Pi appends the new section to the conversation rather than rewriting the prompt ([[context-assembly]]).
- **Context files**: per directory, the first of `AGENTS.override.md` / `AGENTS.md` / `CLAUDE.md` found, from the global agent dir and every ancestor of cwd (`resource-loader.ts` `loadContextFileFromDir`). Nested git worktrees are de-duplicated. These files load even when project trust is declined.
- **Sessions**: JSONL tree with 12 entry types, including `compaction`, `branch_summary`, `context_edit` and `label`. `/tree`, `/fork` and `/clone` handle branching.
- **[[context-compaction]]**: runs automatically when `contextTokens > contextWindow - reserveTokens` (16384 by default) and keeps the most recent 20k tokens. Older turns get a structured LLM summary (Goal / Progress / Key Decisions / Next Steps / Critical Context) that builds on the previous summary. Leaving a branch triggers a separate branch summary. Raw entries are never deleted.
- **Memory**: no built-in long-term store. Memory is the context files, the session tree and the summaries.

## Tools & MCP & sub-agents

- **Tool registry** (`src/core/tools/index.ts` `ToolName`): 8 tools, `read`, `bash`, `powershell`, `edit`, `write`, `grep`, `find`, `ls`. Only `read`, `bash`, `edit` and `write` are active by default (`DEFAULT_TOOL_NAMES`, `settings-manager.ts`).
- **Built-in extensions** (`src/extensions/index.ts`): `llama.cpp`, `codemode`, `tool-search`, `mcp`. The last three can be replaced by a third-party extension. They add `codemode`, `tool_search`, `mcp__<server>__<tool>` tools and 3 MCP resource tools.
- **[[mcp|MCP]]**: stdio and streamable HTTP with OAuth; SSE is rejected. User config lives in `~/.pi/agent/mcp.json`. Project config in `.pi/mcp.json` loads only if the project is trusted. Each server or tool has an **exposure** setting: `codemode` (the default, callable only from scripts), `deferred` (loaded via [[tool-search]]), `direct`, or `hidden`. Results over 20 KB are middle-truncated.
- **Codemode** ([[programmatic-tool-calling]], [[code-as-harness]]): the model writes JavaScript that runs in QuickJS compiled to WASM. Its only capability is calling injected tools. Limits: 256 MB of memory, no timers, 2 workers. It also exposes `models.classify()` and image generation.
- **[[subagents|Sub-agents]]**: still not in core. `examples/extensions/subagent/` spawns isolated `pi` processes.
- **Skills**: the agentskills.io spec ([[agent-skills]], [[skill-md-format]]). Pi lists names and descriptions, loads the full body on demand, and also reads `.agents/skills`. Prompt templates and Pi packages (npm/git) are also supported.

## Hooks & extension points

Extensions are in-process TypeScript loaded via jiti. `ExtensionAPI.on` has **41 event overloads** (`src/core/extensions/types.ts` L1643–1710). Agent core separately exposes 2 tool hooks (`beforeToolCall`/`afterToolCall`) and about 6 more loop callbacks.

| Event | Fires | Block / modify? |
|---|---|---|
| `project_trust` | before project resources load | decides trust (yes/no, remember) |
| `resources_discover` | startup / reload | adds skill, prompt and theme paths |
| `input` | user submits text | `transform` or `handled` (swallow) |
| `before_agent_start` | before a run | inject a message, replace the system prompt |
| `context`, `context_with_system` | before each request | replace messages |
| `before_provider_request` / `_headers` | before the HTTP call | modify payload / headers |
| `tool_call` | before a tool runs (MCP included) | `block` + reason, `terminate` |
| `tool_result` | after a tool runs | rewrite content, `isError`, details |
| `message_end` | assistant message done | replace the message |
| `turn_end`, `agent_before_settle` | turn boundary / before idle | append entries, force `continue` |
| `session_before_{switch,fork,compact,tree}` | session operations | `cancel`; supply a custom compaction or summary |
| `user_bash` | `!` / `!!` shell commands | replace operations or the result |
| `cache_warming_decision` | idle cache refresh | last handler wins |
| the other 26 | session, agent, turn, message, tool-execution, provider-stream, UI and model-select notifications | observe only |

## Guardrails: permissions & sandbox

| Layer | What exists | Where |
|---|---|---|
| Per-tool approval | **none in core**. Pi "does not ask for approval before every tool call". Extensions can gate `tool_call`. | `docs/security.md`, `examples/extensions/permission-gate.ts` |
| Project trust | prompt or saved decision before loading `.pi/` settings, MCP, extensions, skills and system prompts. Default is `ask`; headless runs skip these resources. | `src/core/project-trust.ts`, `trust-manager.ts`, `~/.pi/agent/trust.json` |
| Shell sandbox | none built in. The docs describe Docker, Docker Sandboxes, OpenShell and the Gondolin micro-VM extension. | `docs/containerization.md`, `examples/extensions/gondolin` |
| Script sandbox | QuickJS/WASM VM for codemode scripts only | `packages/codemode` |
| Limits | output truncation, 20 KB MCP cap, failing `length`-truncated calls, stdout guard in RPC/JSON modes | `tools/truncate.ts`, `output-guard.ts` |

The docs treat prompt injection as out of scope. They say isolation, not trust prompts, is the security boundary ([[prompt-injection]]).

## Changes since the earlier wiki pages

- **Tool count**: the old pages say "7 built-in" (or "7 plus `powershell.ts`"). The registry has 8, and only 4 are active by default. Codemode, tool_search and MCP tools now come with built-in extensions.
- **Permissions**: "none in core" still holds for tool calls, but project trust is a new consent gate that the old pages leave out.
- **Hooks**: the 2026-11-02 count of 40 extension events is now 41 type overloads. This may reflect a different counting method rather than a code change. "2 in agent core" is right for tool hooks, but agent core also has `prepareRequest`, `prepareNextTurn`, `finishTurn`, `transformContext` and the queue callbacks.
- **Runtime**: Node ≥22.19, not ≥20. `pi-web-ui` is gone, and 7 new packages exist. The provider count is 43 modules and 10 chat APIs, not "~25 providers, 9 APIs".
- **Repo**: `earendil-works/pi`, not `pi-mono`. Earendil staff such as Armin Ronacher commit to it.
- **Goal command**: still none in core.

## Concepts & entities

[[pi-mono]], [[mario-zechner]], [[openclaw]], [[claude-code]], [[opencode]], [[hermes-agent]], [[mcp]], [[tool-search]], [[programmatic-tool-calling]], [[code-as-harness]], [[context-compaction]], [[context-assembly]], [[context-engineering]], [[subagents]], [[agent-skills]], [[skill-md-format]], [[prompt-injection]], [[agentic-harness-engineering]], [[agent-three-layer-model]].

## References

- Repo: https://github.com/earendil-works/pi @ `f6d0c2f9b6`
- Earlier pages: [[pi-mono-agent-doc-2026]], [[agent-capabilities-check-2026-11]], [[agent-goal-loops-extension-docs-2026]], [[agent-products-repo-history-2026]]
- Wiki syntheses: [[cross-agent-comparison-2026]], [[hooks-in-modern-agents]], [[fundamental-components-of-harness]]
