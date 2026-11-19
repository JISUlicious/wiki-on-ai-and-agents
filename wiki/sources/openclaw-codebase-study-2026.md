---
title: "OpenClaw codebase study (openclaw/openclaw @ 9d6f4a94ad, November 2026)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - openclaw-codebase-study-2026-11-18.md
year: 2026
url: https://github.com/openclaw/openclaw
status: complete
importance: medium
tags:
  - 2026
---

# OpenClaw codebase study (November 2026)

Read on 2026-11-18 from [openclaw/openclaw](https://github.com/openclaw/openclaw) at commit `9d6f4a94ad` (2026-11-18, package version `2026.11.5`). Only source files, a few dev docs and `VISION.md` were read. Nothing was installed, built or run, so this page records what the code says, not how well it works. Channel adapters were skipped. Raw notes with file paths: `sources/openclaw-codebase-study-2026-11-18.md`.

## Summary

[[openclaw|OpenClaw]] is a personal-assistant **Gateway**. Since dropping embedded Pi it has grown into a very large runtime of its own: about 14.8k non-test TypeScript files in `src/`, 25 workspace packages, 187 bundled plugin manifests and 52 bundled skills. The agent loop now lives in OpenClaw's own `@openclaw/agent-core` package. Around it sit 68 catalogued core tools behind default-on [[tool-search]], 43 plugin hooks (several can block or rewrite), an LLM exec reviewer, and **turn taint**: owner-level tools are refused once network content has entered the turn. MCP runs both ways, ACP agents can be embedded, collector-style sub-agent swarms are on by default, and `/goal` drives auto-continuation turns with optional token budgets.

## Architecture

- **Language/runtime**: TypeScript ESM on Node `>=24.16.0 <25 || >=26.1.0` (`package.json` `engines`). There are Rust crates for the node host, the Gateway client and an iroh tunnel, plus native apps for macOS, iOS, Android and Linux.
- **Process model**: one long-lived Gateway process (default port 18789, `src/gateway/server-start.ts`) with worker-thread pools for code mode, compaction planning, file-tool planning and SQLite writes (`@openclaw/worker-runtime`). MCP servers, ACP agents and exec commands run as child processes. Sandboxes are containers or SSH hosts.
- **State**: `~/.openclaw/openclaw.json`, a shared `state/openclaw.sqlite`, and a per-agent `openclaw-agent.sqlite` for auth profiles, sessions and runtime state.
- **Runtimes/harnesses**: the built-in runtime id is `openclaw`, and the legacy alias `pi` normalizes to it. Plugins can register other harnesses (`registerAgentHarness`), for example `codex`, `acpx`, `agentsapi` (OpenAI Agents API) and `copilot`. Only `@earendil-works/pi-tui` remains from Pi.
- **Providers**: `packages/ai` + `packages/llm-core` hold the transport. 76 plugin manifests declare providers and 28 declare channels.

## Agent loop

`packages/agent-core/src/agent-loop.ts` (`runAgentLoop`) has an outer loop for follow-up messages and an inner loop that runs while tool calls or steering messages are pending. Each turn:

1. Commits queued steering.
2. Optionally swaps the system prompt and tools (`prepareContinuation`).
3. Streams the model response. **Tool calls can start while the response is still streaming**, and leftover calls run as a terminal batch.
4. Emits `turn_end`.
5. Lets `prepareNextTurn` switch the model or thinking level.

Tools run in **parallel by default**. Any tool declaring `executionMode: "sequential"` serializes its batch. Steering that arrives mid-batch skips the calls that have not started yet. Async tools deliver results only when the model calls `sessions_yield`.

The run stops on:

- abort
- a provider `error` or `aborted` stop
- a fatal tool failure
- a terminate flag on a tool result
- `shouldStopAfterTurn`
- an empty queue

Guards:

- **Tool-repeat guard** (`tool-repeat-guard.ts`): a note on the 3rd identical call and a `tool_repeat_limit` stop on the 20th.
- **Repetition guard**: catches repeated text and is adapted from Hermes Agent.
- **Run-level retry budget**: 24 + 8 per auth profile, clamped to 32–160 attempts (`embedded-agent-runner/run/helpers.ts`). It covers failover across auth profiles and models, rate limits, idle timeouts, overflow recovery and incomplete-turn recovery.

## Context & memory

- **System prompt** (`src/agents/system-prompt.ts`): there are `full`, `minimal` (sub-agent) and `none` modes. Sections cover Tooling, Skills, Messaging, Workspace, Sandbox, Silent Replies (`NO_REPLY`), Authorized Senders, Runtime and others. A cache-boundary marker splits the stable prefix from the per-run tail.
- **Workspace files** are injected in this order: AGENTS.md, SOUL.md (persona), IDENTITY.md, USER.md, TOOLS.md, BOOTSTRAP.md, MEMORY.md (`system-prompt-context-files.ts`). A per-person `users/<id>/USER.md` overrides the shared USER.md. HEARTBEAT.md drives heartbeat turns.
- **[[context-compaction|Compaction]]**:
  - Trigger: at `min(window − 16384 reserve, 75% of window)`, keeping about 20k recent tokens.
  - Precheck: a check before each request picks `compact_only`, `truncate_tool_results_only` or `compact_then_truncate`.
  - Method: an LLM summary whose prompt also requires keeping sender attribution. Some providers use server-side compaction.
  - Pluggability: the context engine can be swapped (`registerContextEngine`).
- **Memory**: the `memory-core` plugin provides `memory_search`/`memory_get`, backed by SQLite with sqlite-vec plus FTS5/BM25 hybrid search.
  - Before compaction, a memory-flush turn appends notes to `memory/YYYY-MM-DD.md`.
  - "Dreaming" runs `light`/`deep`/`rem` phases that promote snippets to durable memory and write DREAMS.md, a form of [[memory-consolidation]].
  - Alternative memory plugins: `active-memory`, `memory-lancedb`, `memory-wiki`, `gbrain`.

## Tools & MCP & sub-agents

- **Core tools**: 68 ids in `CORE_TOOL_DEFINITIONS` (`src/agents/core-tool-facts.ts`), counted from the catalog description map, which must cover every id. Seven more are runtime-only: `search`, `find`, `grep`, `glob`, `tool_search`, `tool_describe`, `structured_output`. Plugin tools come on top.
  - Profiles: `minimal`, `coding`, `messaging`, `full`.
  - [[tool-search|Tool search]] is on by default, but file and shell tools stay directly visible.
  - Experimental **Code Mode** hides the catalog behind `exec`/`wait` and has the model write JavaScript run in QuickJS/WASM, a form of [[programmatic-tool-calling]].
- **[[mcp|MCP]]**:
  - Client: stdio, SSE and Streamable HTTP, in about 60 `agent-bundle-mcp-*` files.
  - Server: `src/mcp/channel-server.ts` exposes 9 conversation and permission tools, and `openclaw-tools-serve.ts` serves selected built-in tools.
- **[[acp|ACP]]**: `src/acp/` is an ACP server. The `acpx` plugin embeds external coding agents as a harness. `extensions/a2a` is an [[a2a|A2A]] v1.0 channel.
- **[[subagents|Sub-agents]]**: tools are `sessions_spawn`, `subagents`, `sessions_yield`, `agents_wait` and `sessions_send`.
  - Defaults: spawn depth 5, 5 children per agent, 8 concurrent (`src/config/agent-limits.ts`).
  - **Swarm** collector mode is enabled by default, with 32 concurrent children per group.
- **Skills** ([[agent-skills]], [[skill-md-format]]): skills come from bundled, managed, workspace, extra-directory, `~/.agents/skills` and project `.agents/skills` sources. The model uses them through `skills_search`/`skills_read` and can write its own through `skill_workshop`.

## Hooks & extension points

43 plugin hooks are defined in `src/plugins/hook-types.ts` and `hook-model.types.ts`. Modifying hooks run in priority order and their results are merged. Void hooks run in parallel. Hooks fail open unless marked otherwise.

| Hook group | Names | Can block / modify? |
|---|---|---|
| Model | `before_model_resolve`, `before_model_request`, `model_call_started`, `model_call_ended`, `llm_input`, `llm_output` | `before_model_request` injects prompt text and fails closed |
| Agent/turn | `before_agent_run`, `agent_turn_prepare`, `before_prompt_build`, `before_agent_reply`, `before_agent_finalize`, `agent_end`, `run_stopped` | `before_agent_run` fails closed; prompt-build hooks inject text; finalize can replace the answer |
| Tools | `before_tool_call`, `after_tool_call`, `tool_result_persist`, `before_message_write` | `before_tool_call` can rewrite params, block, or `requireApproval`; persist hooks can rewrite or block |
| Compaction/session | `before_compaction`, `after_compaction`, `before_reset`, `session_start`, `session_end` | observational, with 30s budgets |
| Channel/messages | `before_channel_ingress`, `inbound_claim`, `channel_pairing_requested`, `message_received`, `message_sending`, `reply_payload_sending`, `message_sent`, `before_dispatch`, `reply_dispatch` | ingress gate passes or blocks and fails closed; sending hooks modify |
| Sub-agents | `subagent_delivery_target`, `subagent_spawned`, `subagent_progress`, `subagent_ended` | delivery target is modifying |
| Gateway/other | `gateway_start`, `gateway_stop`, `heartbeat_prompt_contribution`, `cron_reconciled`, `cron_changed`, `skill_changed`, `before_install`, `resolve_exec_env` | `before_install` and `resolve_exec_env` modify |

There is also a file-based internal hook system with 15 event keys, such as `command:new`, `session:compact:before` and `gateway:pre-restart` (`src/hooks/internal-hook-types.ts`). The plugin API has 59 `register*`/`on*` members, including harnesses, context engines, trusted tool policies, Gateway access policies and sandbox backends.

## Guardrails: permissions & sandbox

| Mechanism | What the code does | Where |
|---|---|---|
| Exec approvals | security `deny`/`allowlist`/`full`, ask `off`/`on-miss`/`always`; **defaults are `full` + `off`**, with ask fallback `deny` | `src/infra/exec-approvals-config.ts` |
| Exec auto-review | an LLM returns allow, deny or ask with a risk level for each command. Failures map to ask, and injection aimed at the reviewer maps to deny | `src/agents/exec-auto-reviewer.ts` |
| Plugin approvals | `before_tool_call` `requireApproval`: allow-once, allow-always or deny, forwarded to channels | `src/plugins/hook-before-tool-call-result.ts` |
| Turn taint | network-sourced tool output taints the turn, and owner-only tools (sessions, terminal, computer, nodes, ...) are refused | `packages/agent-core/src/turn-taint.ts`, `src/security/dangerous-tools.ts` |
| Untrusted-content wrapping | random-id `EXTERNAL_UNTRUSTED_CONTENT` markers, plus 14 suspicious patterns that are logged, not blocked | `src/security/external-content.ts` |
| Sandbox | mode `off`/`non-main`/`all`, **default off**; built-in docker, podman and ssh; plugin openshell, mxc and crabbox workers | `src/agents/sandbox/` |
| DM policy | `pairing`/`allowlist`/`open`/`disabled`, default `pairing` | channel config schemas |
| Tool policy | profiles plus allow/deny pipeline and sandbox tool policy | `src/agents/tool-policy*.ts` |

This connects to [[indirect-prompt-injection]] and [[prompt-injection]].

## Goal command

- `/goal` supports start, pause, resume, complete, block and clear.
- The model tools are `get_goal`, `create_goal` and `update_goal`. `update_goal` accepts only complete or blocked, and blocked only after three consecutive turns stuck on the same blocker.
- While a goal is active, a continuation turn is queued ("Advance the active goal…"). A token budget moves the goal to `budget_limited`.
- No separate judge was found. Compare [[ralph-loop]] and [[goal-command-lineage-repos-2026]].

## Changes since the earlier wiki pages

- **Runtime**: Pi is fully replaced by `@openclaw/agent-core`, with Hermes-derived repetition guards and harness plugins (Codex, ACPX, Agents API).
- **Node**: ≥22.12 → ≥24.16.
- **Scale**: "39 extensions" → 187 plugin manifests.
- **Hooks**: "~24 observational" → 43 hooks, some blocking or approval-raising. `subagent_spawning` is gone and `subagent_progress` is new.
- **Tools**: "pi's 7 + ~13" → 68 catalog ids plus runtime-only tools, tool search and Code Mode.
- **MCP**: was refused in core. It now has a full client and two server surfaces.
- **Sub-agents**: May wording discouraged agent hierarchies. Swarm collector mode is now on by default, and VISION now only rejects "heavy orchestration layers" that duplicate existing ones.
- **Memory**: no longer only Markdown. It is SQLite hybrid search with dreaming consolidation.
- **Sandbox**: has added MXC and Crabbox next to Docker, Podman, SSH and OpenShell.
- **Goal**: confirmed as model or operator completion, with auto-continuation and token budgets.

## Concepts & entities

[[openclaw]] · [[pi-mono]] · [[hermes-agent]] · [[openai-codex]] · [[claude-code]] · [[mcp]] · [[acp]] · [[a2a]] · [[subagents]] · [[tool-search]] · [[programmatic-tool-calling]] · [[context-compaction]] · [[memory-consolidation]] · [[agent-skills]] · [[indirect-prompt-injection]] · [[ralph-loop]] · [[hooks-in-modern-agents]] · [[agent-products-repo-history-2026]] · [[agent-capabilities-check-2026-11]] · [[pi-codebase-study-2026]] · [[hermes-agent-codebase-study-2026]]

## References

- Repository: https://github.com/openclaw/openclaw @ `9d6f4a94ad` (read 2026-11-18; nothing executed)
- Raw notes: `sources/openclaw-codebase-study-2026-11-18.md`
- Earlier pages: [[openclaw-agent-doc-2026]], [[cross-agent-comparison-doc-2026]]
