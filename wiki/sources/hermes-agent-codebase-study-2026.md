---
title: "Hermes Agent — Codebase Study (2026-11)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - hermes-agent-codebase-study-2026-11-18.md
arxiv_id: null
venue: GitHub source code (NousResearch/hermes-agent @ 0e8e421640)
authors:
  - Nous Research contributors
year: 2026
url: https://github.com/NousResearch/hermes-agent
tags:
  - 2026
status: complete
importance: medium
---

# Hermes Agent — Codebase Study (2026-11)

**Repo read**: `NousResearch/hermes-agent` @ `0e8e421640` (commit date 2026-11-18). It was shallow-cloned and read only. Nothing was built, installed, tested or run. Tool and hook counts come from small AST scripts run over the source, outside the clone. Raw notes: `sources/hermes-agent-codebase-study-2026-11-18.md`.

## Summary

[[hermes-agent|Hermes Agent]] is [[nous-research|Nous Research]]'s Python personal agent. One agent core serves a classic CLI, an Ink TUI, an Electron desktop app, a web dashboard, [[acp]] editors and a messaging gateway with 26 platform plugins. Since May the code has been heavily refactored. `run_agent.py` went from about 15.7k lines to 1,621 and is now a facade. The turn loop is split into 33 `agent/turn_*.py` phase modules. Two rules in the root `AGENTS.md` drive the design. **Per-conversation [[prefix-caching|prompt caching]] is sacred**: compression is the only allowed cache break. **The core is a narrow waist**: new capability should arrive as a skill, plugin or MCP server, not a new core tool. A CI snapshot test fails on any change to the stock system prompt or `tools[]`.

## Architecture

- **Language/runtime**: Python ≥3.11,<3.15 (uv). TypeScript for `ui-tui/` (Ink fork), `apps/desktop/` (Electron 44) and `web/`.
- **Process model**: a per-profile gateway-owner process (`hermes gateway run`, started on demand) holds sessions, agents and tools. The TUI, desktop app and dashboard attach over WebSocket (`hermes-gateway-v1`). The classic CLI can still run the agent in-process. One process can serve many profiles, and secrets are profile-scoped (`agent/secret_scope.py`).
- **Packages**: `agent/` (loop, prompt, compression, providers), `tools/` (registry and tools), `hermes_cli/` (CLI, config, plugins, goals), `gateway/`, `tui_gateway/`, `persistence/` (SQLite + FTS5 storage interface; `hermes_state.py` is now a deprecation shim), `plugins/`, `cron/`, `acp_adapter/`.
- **Providers**: 39 provider-profile plugins in `plugins/model-providers/`. Besides chat completions, `api_mode` covers native Anthropic, Bedrock, Gemini and Vertex adapters, Codex Responses, and an opt-in `codex_app_server` mode that runs the whole turn in a Codex subprocess. Credential pools, `fallback_providers`, and per-task auxiliary models are also supported.

## Agent loop

`AIAgent.run_conversation` takes a session turn lease and calls `agent/conversation_loop.py::_run_conversation_turn`. The loop is synchronous: `while (api_call_count < max_iterations and iteration_budget.remaining > 0) or _budget_grace_call`. Each iteration runs phases that return a verdict (`return`/`break`/`continue`): prepare, assemble request, preflight gate, API retry loop (`api_max_retries: 3`, then the fallback chain and an auto-recover ladder), normalize response, then `run_tool_round` or `finish_text_response`.

- **Stop conditions**: a text-only answer; budget exhaustion (plus one grace call); interrupt; a `pre_llm_call` veto; stop gates; a **verification nudge** when the model tries to finish right after editing code with no fresh check (`agent/verification_stop.py`, `max_verify_nudges: 3`); repetition and tool-loop guards.
- **Iteration cap**: the default is now unlimited. `run_agent.py` sets `max_iterations = sys.maxsize`, and config `agent.max_turns: null`; its comment says caps caused "silent mid-task truncation". `agent/AGENTS.md` still says 500, which is stale. Subagents default to 250.
- **Parallel tools**: `_plan_tool_batch_segments` splits a batch into ordered parallel and sequential runs. Read-only tools run in parallel. File readers and writers run in parallel only when their paths don't overlap. MCP tools run in parallel only if their server opts in. Interactive tools act as barriers. Up to 8 workers, with approval prompts serialized.

## Context & memory

- **System prompt** (`agent/system_prompt.py`) is built once per session in three tiers (stable, context, volatile) and restored byte-identical on later turns. Mid-conversation additions go into user messages or tool results (skill commands, subdirectory `AGENTS.md` hints, `/steer`).
- **Instruction files**: `SOUL.md` (identity) plus **one** project file type, whichever is found first: `.hermes.md`/`HERMES.md` → `AGENTS.md` chain → `CLAUDE.md` → `.cursorrules`. They are scanned for [[prompt-injection]], and a hit replaces the file with a BLOCKED notice.
- **[[context-compaction|Compaction]]**: the gateway runs session hygiene at 85%. The agent's `ContextCompressor` triggers at 50% (floored at 75% for windows under 512K). It first prunes old tool results without an LLM call, then writes a structured summary with an auxiliary model. There is a deterministic fallback summary, manual `/compress`, and a `post_compaction` hook.
- **Memory**: (1) frozen `MEMORY.md`/`USER.md` snapshot via the `memory` tool; (2) memory-provider plugins (only byterover and retaindb in-tree; honcho, mem0 and others are now catalog plugins); (3) `session_search` over FTS5; (4) a learning loop. That loop has a post-turn **background review** fork (`agent/background_review.py`) that decides whether to save skills or memory, and an inactivity-triggered skill **curator** (`agent/curator.py`) that archives but never deletes and keeps a rollback ledger. See [[tiered-memory]] and [[self-improving-agent]].

## Tools, MCP & sub-agents

- **Tools**: 68 built-ins are registered statically in `tools/` (45 with literal names, plus 23 registered in loops: kanban ×15, discord ×2, feishu_drive ×4, xai_video ×2). The default `_HERMES_CORE_TOOLS` list has 41. Plugins (a2a, google_meet) and MCP servers add more. Gated by `check_fn`, so the live count varies. `tools/tool_search.py` does progressive disclosure: MCP and plugin tools are hidden behind `tool_search`/`tool_describe`/`tool_call` bridges ([[tool-search]]). `execute_code` provides [[programmatic-tool-calling]].
- **[[mcp|MCP]]**: client over stdio, SSE and streamable HTTP, with OAuth (including device flow), elicitation and sampling. 66 curated server configs live in `optional-mcps/`. "Hermes as an MCP server" left core and is now the `mcp-serve` plugin.
- **[[subagents|Sub-agents]]**: `delegate_task` spawns a child `AIAgent` (single, batch-parallel or async/background). Roles are leaf and orchestrator. Default depth is 1 (`delegation.max_spawn_depth` raises it), with up to 10 concurrent children. Children can't delegate further, ask the user, write memory, send messages or schedule cron. Their approvals auto-deny by default. Delegation backends are pluggable. A kanban board with a dispatcher spawns worker agents per task.
- **[[agent-skills|Skills]]**: [[skill-md-format|SKILL.md]] files: 44 bundled and 141 optional. The agent writes skills with `skill_manage`. External skills pass through `skills_guard` scanning with builtin, trusted and community trust levels.

## Hooks & extension points

| Mechanism | Where | What it can do |
|---|---|---|
| 48 plugin hooks (`VALID_HOOKS`) | `hermes_cli/plugins.py` | Mostly observers. Exceptions: `pre_tool_call` can block or modify args; `pre_llm_call` can inject context or veto the turn; `pre_verify` can force the agent to continue (also accepts the Claude Code Stop shape); `pre_gateway_dispatch` can skip or rewrite; the `transform_*` hooks can replace LLM output, tool results, terminal output or error classification; `pre_transcription` can mutate the request |
| Middleware | `hermes_cli/middleware.py` | `tool_request`, `tool_execution`, `llm_request`, `llm_execution`: rewrite a request or wrap execution |
| Shell hooks | `agent/shell_hooks.py` | `hooks:` in config run shell commands with JSON on stdin. Exit code 2 blocks `pre_tool_call` (Claude Code/Cursor convention). Each command needs consent on first use and fails open unless `fail_closed` is set |
| Outbound webhooks | `agent/outbound_webhooks.py` | Subscribe to any hook event |
| Plugin API | `hermes_cli/plugins_context.py` | ~30 `register_*` methods: tools, commands, platforms, memory/context engines, prompt sections, approval transports, redaction patterns, terminal backends, delegation, providers |
| Plugin catalog | `plugin-catalog/` | 2,451 entries, human-merged and pinned to exact 40-character SHAs, with no self-updaters allowed |

The 48 hooks: the session lifecycle ×7, `pre/post_tool_call`, `pre/post_llm_call`, `pre/post_api_request`, `api_request_error`, `pre/post_auxiliary_call`, stream ×3 plus `on_interim_message`, `transform_*` ×4, `pre_verify`, `subagent_start/stop`, `on_skill_lifecycle`, approval ×2, human input ×2, gateway ×3 (`pre_gateway_dispatch`, `post_gateway_admission`, `gateway_platform_event`), `pre_command`, `agent_loop_stopped`, `on_room_member_activity`, `pre_transcription`, kanban ×8, `post_compaction` and `post_curator_run`.

## Guardrails: permissions & sandbox

| Layer | Detail |
|---|---|
| Approval modes | `approvals.mode`: `manual` (default), `smart` (an auxiliary-LLM guardian returns APPROVE/DENY/ESCALATE and treats the command as untrusted; credited to Codex Smart Approvals) or `off`. Per-session `/yolo`. Answers can be once, session or always (persisted allowlist). Includes a denial breaker |
| Command checks | Hardline patterns (system-dir `rm -r`, block-device writes, fork bombs) can't be approved even under YOLO, and neither can a command the scanner couldn't finish parsing. Dangerous patterns prompt; user deny rules always apply. Unattended contexts (cron, `-q`, webhooks) use their configured deny/approve setting |
| Other gates | `execute_code` approval, protected instruction-file writes (SOUL.md, config), self-repo, path and URL safety, plugin-escalated approval that fails closed |
| Sandboxes | 6 in-tree terminal backends (local, docker, modal, daytona, vercel_sandbox, ssh). Singularity is now a catalog plugin, and plugins can add more. Container backends skip the command guards unless host paths are bind-mounted. Docker egress control |
| Secrets | `agent/redact.py` with about 20 pattern leaves, a log formatter, profile-scoped `get_secret`, vault backends; `.env` holds secrets only |
| Untrusted input | Injection scan on context files; webhook sessions get only web, vision and clarify tools; optional `security-guidance` plugin (25 regex warnings ported from Anthropic's plugin) |

See [[guardrailing]] and [[indirect-prompt-injection]].

## Goal command

`/goal` (`hermes_cli/goals.py`) is described in its source as "the Ralph loop for Hermes" ([[ralph-loop]]). After each turn, shell **quality gates** run first. A failing gate's output becomes the next prompt and the judge is skipped. If all gates pass, an auxiliary-model **judge** ([[agent-as-a-judge]]) returns done, continue or wait. The defaults are 20 turns, fail-open on judge errors, and an auto-pause after repeated parse or transport failures. `/subgoal` adds criteria. Related: `/loop` (Claude Code parity, `LOOP_COMPLETE`, `max_ticks: 100`), `/heartbeat`, `hermes cron`.

## Changes since the earlier wiki pages

- The loop moved from a ~15.7k-line `run_agent.py` into 33 `agent/turn_*.py` phase modules. `cli.py` is ~1.8k lines (was ~13.5k).
- Sandbox backends went from 7 to 6 in-tree. Singularity moved to the catalog.
- Hooks went from 46 to 48, and Claude Code-style shell hooks and four middleware kinds were added.
- The default iteration cap is now unlimited (the old docs said 500).
- Memory providers other than byterover and retaindb moved out to catalog plugins.
- Session storage moved to `persistence/`; `hermes_state.py` is deprecated and will be removed in v0.27.0.
- The `smart` approval guardian, `tool_search` progressive disclosure, kanban workers, `/loop` and `/heartbeat`, `/moa` and the Codex app-server mode are all in the code. Of these, only `/loop`, `/heartbeat` and kanban were mentioned on earlier pages.

## Concepts & entities

[[hermes-agent]] · [[nous-research]] · [[claude-code]] · [[openai-codex]] · [[opencode]] · [[pi-mono]] · [[openclaw]] · [[zed]] · [[acp]] · [[mcp]] · [[subagents]] · [[agent-skills]] · [[skill-md-format]] · [[context-compaction]] · [[prefix-caching]] · [[tiered-memory]] · [[self-improving-agent]] · [[tool-search]] · [[programmatic-tool-calling]] · [[ralph-loop]] · [[agent-as-a-judge]] · [[guardrailing]] · [[prompt-injection]] · [[indirect-prompt-injection]] · [[agentic-harness-engineering]] · [[cross-agent-comparison-2026]] · [[hooks-in-modern-agents]] · [[fundamental-components-of-harness]]

## References

- Repo: https://github.com/NousResearch/hermes-agent @ `0e8e421640`. Key files: `AGENTS.md`, `agent/AGENTS.md`, `agent/conversation_loop.py`, `agent/tool_dispatch_helpers.py`, `agent/system_prompt.py`, `agent/prompt_builder.py`, `hermes_cli/plugins.py`, `hermes_cli/middleware.py`, `agent/shell_hooks.py`, `tools/approval.py`, `tools/terminal_tool_backends.py`, `toolsets.py`, `tools/delegate_tool*.py`, `hermes_cli/goals.py`, `hermes_cli/config_defaults.py`.
- Earlier wiki sources: [[hermes-agent-doc-2026]], [[agent-goal-loops-extension-docs-2026]], [[agent-products-repo-history-2026]].
