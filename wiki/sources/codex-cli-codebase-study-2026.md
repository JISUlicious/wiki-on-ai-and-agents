---
title: "Codex CLI codebase study (openai/codex @ 4389a526d5)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - codex-cli-codebase-study-2026-11-18.md
url: https://github.com/openai/codex
venue: Code-level study of the open-source repository
authors:
  - "OpenAI (repository); notes by the wiki maintainer"
year: 2026
tags:
  - 2026
status: complete
importance: high
---

# Codex CLI codebase study (2026)

**Raw notes**: `sources/codex-cli-codebase-study-2026-11-18.md`
**Read**: `openai/codex` at commit `4389a526d5` (2026-11-18). Source was read only. Nothing was built, installed or run.

## Summary

[[codex-cli]] is [[openai|OpenAI]]'s open-source coding agent. The live code is `codex-rs/`, a Rust/Tokio Cargo workspace of about 150 crates and about 2.4M lines. `codex-cli/` is no longer the old TypeScript agent: it is only an npm launcher (`bin/codex.js`) that `execve`s the native binary. Every surface (TUI, `codex exec`, IDE, Desktop app, SDKs) talks to one JSON-RPC **app-server**, either in-process or as a daemon. The harness speaks only the [[responses-api]] wire format. It sends tool calls to run while the model is still streaming, compacts context in three different ways, and can route approvals to an LLM "Guardian" reviewer instead of the user. Hook names and plugin manifests are deliberately compatible with [[claude-code]].

## Architecture

- **Entry**: `codex` binary (`cli/src/main.rs`, ~30 subcommands: `exec`, `review`, `mcp`, `plugin`, `app-server`, `sandbox`, `execpolicy`, `cloud`, `resume`, `fork`, `exec-server`, ...). `arg0/` re-dispatches on argv[0] for `apply_patch` and sandbox helpers.
- **App-server hub**: `tui/Cargo.toml` depends on `codex-app-server-client`, not on `codex-core`. `app-server/src/in_process.rs` runs the same protocol over in-memory channels. `app-server-daemon/` keeps a shared long-lived daemon.
- **Exec-server**: a JSON-RPC PTY process server, used to run tools in remote or other environments (`wait_for_environment`).
- **Providers**: built in are `openai`, `amazon-bedrock`(+runtime), `gcp`, `ollama` and `lmstudio`. The `chat` wire API was removed, so Responses only. WebSocket is used where possible, with HTTPS fallback.
- **Model catalog as policy**: `models-manager/models.json` carries settings for each model: prompt template, `shell_type`, `apply_patch_tool_type`, `auto_compact_token_limit`, `multi_agent_version`, `supports_parallel_tool_calls`.
- **SDKs**: `sdk/typescript` spawns the CLI and exchanges JSONL. There is also a Python SDK.

## Agent loop

`core/src/session/turn.rs` `run_turn`:

1. Drain the results of async hooks, then compact before sampling if the context is over the limit.
2. Loop: drain steered user input, run `UserPromptSubmit`, capture a `StepContext` (tools, environments, MCP servers named by mentions), record a world-state diff, then `run_sampling_request`.
3. While streaming, each finished tool-call item is pushed into a `FuturesOrdered` in-flight queue, so tools start before the response ends. `tools/parallel.rs` gates them with an `RwLock`: parallel-safe tools share it and the rest run exclusively. The prompt always sets `parallel_tool_calls: true`.
4. If there were tool calls or pending input, compact mid-turn if needed and loop again. Otherwise run `Stop` hooks. A blocking Stop hook with a prompt injects that prompt and continues the turn.

Retries: stream 5 and request 4 by default, capped at 100, with a 300 s idle timeout and WebSocket→HTTPS fallback. **No max-turn counter exists.** The loop is bounded by a `RolloutBudget` work gate (spend units shared across a thread tree), usage limits, cancellation and hooks.

## Context & memory

- **Base prompt** comes from the model's catalog template, with fallback to `models-manager/prompt.md`. Developer fragments add permissions and approval rules, writable roots, the skills/plugins/apps catalogs and multi-agent guidance.
- **AGENTS.md**: `AGENTS.override.md` / `AGENTS.md` are gathered from the project root (marker `.git`) down to cwd and concatenated, with a 32 KiB budget. They are skipped for untrusted projects. See [[project-memory]].
- **[[context-compaction]]** triggers at min(configured limit, 90% of the window), checked before the turn, mid-turn and after the turn. There are three methods: (a) local LLM summarization ("CONTEXT CHECKPOINT COMPACTION" handoff prompt, keeping ≤20k tokens of user messages); (b) provider-side remote compaction v2; (c) token-budget mode, which opens a fresh window, paired with the `new_context` and `get_context_remaining` tools. Compaction can also start early and run asynchronously.
- **Memories**: a background two-phase pipeline. Phase 1 has the model extract memories from past rollouts into SQLite, with secrets redacted. Phase 2 has a sandboxed consolidation sub-agent merge them into `~/.codex/memories` (git-tracked), with no network and no approvals. The model reads them through the `memories.list/read/search/add_ad_hoc_note` tools.

## Tools, MCP, sub-agents, skills

- **Built-in tools** (counted from `core/src/tools/spec_plan.rs` plus tool-name constants; nearly all are gated by feature or model): **22 core** (`exec_command`, `write_stdin`, `apply_patch`, `update_plan`, `view_image`, `request_user_input`(+async), `send_message_to_user_async`, `request_permissions`, `new_context`, `get_context_remaining`, `curr_time`, `sleep`, 2 plugin-install tools, `wait_for_environment`, 3 MCP-resource tools, `tool_search`, code-mode `exec`/`wait`). Add **5–6 multi-agent** tools (v1 or v2), hosted `web_search` and image generation, and about 15 extension tools (goal, memories, skills, web, imagegen, ...).
- **Code mode**: the model writes JS that runs in an embedded V8 isolate and calls tools as functions. Compare [[programmatic-tool-calling]].
- **[[tool-search]]**: MCP and extension tools can be deferred behind search.
- **[[mcp]]**: client via `rmcp-client` + `codex-mcp` (OAuth, elicitation, resources, tool cache). Tools are named `mcp__server__tool`. No `codex mcp-server` subcommand was found at this commit. No [[acp]] support was found.
- **[[subagents]]**: `spawn_agent`/`send_input`/`wait_agent`/`close_agent`/`resume_agent` (v1) or `spawn_agent`/`send_message`/`followup_task`/`wait_agent`/`interrupt_agent`/`list_agents` (v2). Built-in roles are `default`, `explorer`, `worker` and `awaiter`; custom roles come from TOML. Defaults: 6 threads per session, depth 1. Roles can only narrow the parent's authority.
- **[[agent-skills]]**: `SKILL.md` in `.codex/skills`, `.agents/skills`, `~/.codex/skills` and `~/.agents/skills`, plus admin, system-bundled (`skill-creator`, `skill-installer`, `imagegen`, ...) and plugin roots. See [[skill-md-format]].

## Hooks & extension points

A Claude-Code-style hook engine (the type is literally `ClaudeHooksEngine`) with **14 events**. Handlers can be `command` or `mcp_tool`. `prompt` and `agent` handlers parse but are skipped as "not supported yet". Sources are `hooks.json`, `[hooks]` in config layers, and plugin-bundled hooks, which carry a trust fingerprint.

| Event | Can block / modify |
|---|---|
| `SessionStart`, `SubagentStart` | add context |
| `UserPromptSubmit` | block, add context |
| `PreModelRequest` / `PostModelRequest` | observe (MCP post-hook may price usage) |
| `PreToolUse` | allow/deny/ask, `updatedInput`, context |
| `PermissionRequest` | allow/deny, `updatedInput`, `updatedPermissions`, interrupt |
| `PostToolUse` | block, context, `updatedMCPToolOutput` |
| `PreCompact` / `PostCompact` | observe (block unsupported) |
| `Stop` / `SubagentStop` | block + prompt → continue |
| `Interrupt`, `SessionEnd` | observe |

There is also a legacy `notify` after-agent command. Separately, `ext/extension-api` defines about 20 **compile-time Rust contributor traits** (thread/turn/tool lifecycle, context, instructions, MCP servers, approval review, model-request interceptors) that the in-tree `ext/*` crates implement. **Plugins** (`.codex-plugin/plugin.json`, also reading `.claude-plugin/plugin.json`) bundle skills, MCP servers, apps and hooks, and install from marketplaces or npm. See [[hooks-in-modern-agents]].

## Guardrails: permissions & sandbox

| Layer | Mechanism |
|---|---|
| Approval policy | `untrusted`, `on-request`, `granular` (per category), `never` |
| Approval reviewer | `user` (default) or `auto_review`: LLM Guardian sub-agent with a risk framework |
| Sandbox mode | `read-only`, `workspace-write`, `danger-full-access` (+ external); network off by default |
| macOS | Seatbelt `.sbpl` profiles |
| Linux | bubblewrap (bundled) for FS + `no_new_privs` + seccomp; Landlock legacy |
| Windows | restricted token + ACLs + WFP firewall; MXC variant |
| Network | managed proxy: domain policy, per-request approval, secret substitution |
| Command rules | Starlark `execpolicy` `prefix_rule` allow/prompt/forbidden; managed `prompt-user` checkpoints |
| Secrets | regex `redact_secrets`; proxy-held credentials; `.env` cannot set `CODEX_*` |

`tools/orchestrator.rs` runs approval, picks the sandbox and retries after a sandbox denial. Any escalation needs a fresh review. The Guardian policy prompt tells the reviewer to ignore untrusted content that makes claims about how sensitive data is. This is the most explicit [[prompt-injection]] defence found. See [[guardrailing]].

## Distinctive points

- **One protocol for all surfaces** (app-server). The TUI is a client, not the engine.
- **Tools run during streaming**, with RwLock-based parallelism.
- **Model metadata steers the harness** (prompt, tool shapes, compaction limit).
- **Three compaction strategies**, with speculative async compaction.
- **LLM Guardian** as an approval route, and a **secret-substituting network proxy**.
- **Claude Code compatibility**: hook names and JSON, `.claude-plugin` manifests.
- **`/goal`** is the in-tree `ext/goal` extension: `create_goal`/`get_goal`/`update_goal`, continuing when the thread goes idle. See [[goal-command-lineage-repos-2026]].
- **Review mode** (`tasks/review.rs`, `codex review`) runs a sub-conversation with a review prompt. **Cloud tasks** come via `codex cloud`.

## Concepts & entities

[[codex-cli]] · [[openai]] · [[claude-code]] · [[responses-api]] · [[context-compaction]] · [[project-memory]] · [[subagents]] · [[agent-skills]] · [[skill-md-format]] · [[mcp]] · [[acp]] · [[tool-search]] · [[programmatic-tool-calling]] · [[guardrailing]] · [[prompt-injection]] · [[agentic-harness-engineering]] · [[hooks-in-modern-agents]] · [[cross-agent-comparison-2026]]

## References

- Repository: https://github.com/openai/codex (commit `4389a526d5`, 2026-11-18)
- [[codex-cli-o3-o4-mini-openai-2025]] (launch announcement)
- [[goal-command-lineage-repos-2026]]
- [[openai-codex-skills-2025]]
