---
title: Codex CLI
type: entity
created: 2026-11-01
updated: 2026-11-18
sources:
  - codex-cli-o3-o4-mini-openai-2025.md
  - codex-cli-codebase-study-2026-11-18.md
release_date: 2025-04-16
status: draft
importance: medium
tags:
  - product
---

# Codex CLI

[[openai|OpenAI]]'s local terminal coding agent. It was announced on 16 April 2025 as an experiment alongside the o3 and o4-mini launch. See [[codex-cli-o3-o4-mini-openai-2025]]. Compare [[claude-code]].

**Goals:** `/goal` shipped in 0.128.0 (2026-04-30) and now lives in the in-tree `codex-rs/ext/goal` crate. The working model marks completion after an audit prompt. See [[goal-command-lineage-repos-2026]]. At commit `4389a526d5` the extension exposes `create_goal`, `get_goal` and `update_goal` tools and continues the thread from an `on_thread_idle` callback.

## Code-level overview (2026-11-18)

From a read-only study of `openai/codex` at `4389a526d5`; details and file paths are in [[codex-cli-codebase-study-2026]].

- **Architecture**: Rust/Tokio workspace `codex-rs/` (~150 crates). The old TypeScript `codex-cli/` is now only an npm launcher for the native binary. The TUI, `codex exec`, the IDE, the Desktop app and the SDKs all use one JSON-RPC **app-server** protocol, either in-process or through a daemon. The TUI does not link `codex-core` directly. Providers are Responses-API only (`openai`, Bedrock, GCP, Ollama, LM Studio). A bundled model catalog sets settings for each model: prompt template, shell and patch tool shape, compaction limit.
- **Loop**: `core/src/session/turn.rs` `run_turn`. Tool calls start while the response is still streaming, with an `RwLock` gate for parallel-safe tools. The loop continues while there are tool calls or steered input. `Stop` hooks can force a continuation. There is no max-turn cap: a spend-unit `RolloutBudget`, usage limits and cancellation bound it instead. Compaction triggers at min(config, 90% of the window) and uses one of three methods (local summary, remote v2, or a fresh token-budget window), with speculative async compaction.
- **Context**: `AGENTS.override.md`/`AGENTS.md` from the project root to cwd (32 KiB budget, skipped for untrusted projects). [[agent-skills]] come from `.codex/skills` and `.agents/skills`. A two-phase background memory pipeline consolidates memories into `~/.codex/memories`.
- **Tools**: 22 core tools plus 5–6 multi-agent tools, plus hosted web search and image generation, plus about 15 extension tools, almost all gated by flags. Code mode lets the model run JS in V8 that calls tools. There is an [[mcp]] client and [[tool-search]]. [[subagents]] use the `default`/`explorer`/`worker` roles, with 6 threads and depth 1 by default.
- **Hooks**: 14 Claude-Code-compatible events (`SessionStart`, `UserPromptSubmit`, `Pre/PostModelRequest`, `PreToolUse`, `PermissionRequest`, `PostToolUse`, `Pre/PostCompact`, `SubagentStart/Stop`, `Stop`, `Interrupt`, `SessionEnd`), with `command` and `mcp_tool` handlers. There are also about 20 compile-time Rust extension traits, and plugins in `.codex-plugin` or `.claude-plugin` format. See [[hooks-in-modern-agents]].
- **Guardrails**: approval policy `untrusted`/`on-request`/`granular`/`never`, with an optional LLM "Guardian" auto-reviewer. Sandbox modes are `read-only`/`workspace-write`/`danger-full-access`, enforced by Seatbelt on macOS, bubblewrap plus seccomp on Linux, and restricted tokens plus WFP on Windows. A managed network proxy substitutes secrets, and Starlark `execpolicy` holds the command rules. See [[guardrailing]].
