---
title: OpenClaw
type: entity
created: 2026-05-14
updated: 2026-11-18
sources:
  - openclaw.md
  - comparison.md
  - openclaw-codebase-study-2026-11-18.md
family: agent-product
parameters: "n/a (own @openclaw/agent-core runtime; 76 provider plugins)"
release_date: 2026-01-30
status: complete
importance: medium
tags:
  - product
---

# OpenClaw

> [!warning] Contradiction
> This page, written from May 2026 docs, says OpenClaw embeds pi-mono as its runtime. Commit `bb46b79d3c1` (2026-05-27) internalized that runtime. `THIRD_PARTY_NOTICES.md` now says portions were adapted from Pi, and only `pi-tui` remains a dependency. Earlier names were Warelay, Clawdis, Clawdbot and Moltbot. See [[agent-products-repo-history-2026]]. The 2026-11-18 code read ([[openclaw-codebase-study-2026]], commit `9d6f4a94ad`) confirms that the loop now lives in OpenClaw's own `packages/agent-core`. The legacy runtime id `pi` normalizes to `openclaw`.

OpenClaw is a single-operator, multi-channel personal AI assistant. It runs as a long-lived local **Gateway** daemon plus a fleet of optional client surfaces, all connecting back to the Gateway over WebSocket. Editorial stance from `VISION.md`:

> "OpenClaw is the AI that actually does things. It runs on your devices, in your channels, with your rules."

## Surfaces

- CLI (`openclaw …`) and TUI (built on `pi-tui`)
- Lit-based web Control UI
- Native apps: macOS menu-bar app, iOS, Android, and Linux (`apps/`). There is also a Rust node host (`crates/openclaw-node-host`).
- Messaging-channel plugins: WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Matrix, Teams, Zalo, ... (28 plugin manifests declare channels as of 2026-11-18)
- MCP server and ACP server surfaces (see below)

## Architecture

- **pnpm TypeScript monorepo**. As of 2026-11-18 it has about 14.8k non-test TS files in `src/`, 25 workspace packages, 187 bundled plugin manifests and 52 bundled skills. Node `>=24.16.0 <25 || >=26.1.0`.
- **One Gateway daemon per host**, with `ws://127.0.0.1:18789` as the control plane (`src/gateway/server-start.ts`). Worker-thread pools handle code mode, compaction planning, file tools and SQLite writes.
- **Agent runtime**: `@openclaw/agent-core` (`packages/agent-core/src/agent-loop.ts`). Tool calls start while the response is still streaming, tools run in parallel by default, and there are tool-repeat and text-repetition guards. The run-level retry and failover budget lives in `src/agents/embedded-agent-runner/`. Plugins can register other harnesses (Codex, ACPX, OpenAI Agents API, Copilot).
- **Tools**: 68 core catalog tools plus 7 runtime-only names (`src/agents/core-tool-facts.ts`), with default-on tool search and experimental Code Mode (QuickJS/WASM).
- **Sandbox**: off by default (`mode: off | non-main | all`). Built-in backends are Docker (`openclaw-sandbox:trixie-slim`, plus common/browser image tiers), Podman and SSH. Plugins add OpenShell, MXC and Crabbox cloud workers.
- **Permissions**: exec approvals (security `deny`/`allowlist`/`full`, ask `off`/`on-miss`/`always`; default `full` + `off`), an LLM exec auto-reviewer, `before_tool_call` plugin approvals, tool profiles with allow/deny policy, `/elevated`, and turn-taint refusal of owner-only tools.
- **MCP**: refused in core as of May 2026, using a `mcporter` bridge. As of 2026-11-18 there is a full MCP client (stdio, SSE, Streamable HTTP) and two server surfaces: a channel server with 9 tools, and a built-in tools server. See [[agent-capabilities-check-2026-11]].
- **ACP**: server-side in `src/acp/`; client-side via the `acpx` harness plugin, which can *embed* Codex, [[claude-code|Claude Code]] and other ACP agents from inside OpenClaw. There is also an A2A v1.0 channel plugin.
- **Hooks**: 43 plugin hooks plus 15 internal file-hook events. See [[openclaw-codebase-study-2026]].

> [!warning] Contradiction
> The May 2026 page said "Tools: pi's 7 + OpenClaw's ~13" and "39 extensions, 52 skills" ([[openclaw-agent-doc-2026]]). The 2026-11-18 code has 68 catalogued core tools and 187 plugin manifests. The skill count is unchanged at 52 ([[openclaw-codebase-study-2026]]).

> [!warning] Contradiction
> [[hooks-in-modern-agents]] describes OpenClaw's hooks as "~24 named events", "observational (synchronous handlers)". At commit `9d6f4a94ad` there are 43 hooks (`src/plugins/hook-types.ts`). `before_tool_call` can rewrite params, block, or require operator approval. `before_agent_run`, `before_channel_ingress` and `before_model_request` fail closed. `subagent_spawning` no longer exists.

## Distinguishing features

- **Single-operator, multi-channel**: the "personal AI assistant" form factor done well. Most other documented 2026 agents are coding-focused.
- **Owns its runtime now**: it embedded pi-mono from 2025-12 to 2026-05. The Pi-derived loop has since been rewritten into a large in-house runtime with harness plugins.
- **Channel pluginization**: each messaging channel (WhatsApp, Telegram, etc.) is an in-process plugin that translates platform events into Gateway protocol messages. DMs default to `pairing`.
- **Turn taint**: tool output from the network marks the turn tainted. Owner-level tools are then refused unless the sender is the owner (`src/security/dangerous-tools.ts`). This is a code-level defence against [[indirect-prompt-injection]].
- **Memory dreaming**: the memory-core plugin runs `light`/`deep`/`rem` consolidation phases over Markdown memory, with SQLite hybrid search.
- **Sub-agents**: `sessions_spawn` plus swarm collector mode, on by default (depth 5, 5 children per agent).

> [!warning] Contradiction
> The May 2026 page said OpenClaw "refuses hierarchical multi-agent frameworks" ([[openclaw-agent-doc-2026]]). In 2026-11 code, swarm collector-mode sub-agents are enabled by default (`src/agents/subagents/swarm/swarm-config.ts`). `VISION.md` now rejects only "heavy orchestration layers that duplicate existing agent and tool infrastructure".

**Goal and trigger features:** `/goal` (2026-05-29) lets the working model or operator mark completion. As of 2026-11-18, an active goal queues auto-continuation turns, and an optional token budget moves it to `budget_limited`. `update_goal` accepts only `complete` or `blocked` (`src/auto-reply/reply/goal-continuation.ts`, `src/agents/tools/goal-tools.ts`). No separate judge was found. Standing orders, standing intents, heartbeats and cron automations cover proactive work. See [[agent-goal-loops-extension-docs-2026]].

## References

- [[openclaw-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]
- [[openclaw-codebase-study-2026]] — code read at `9d6f4a94ad`, 2026-11-18

## Related 2026 sources

- [[illicit-distillation]] — named twice in Anthropic's 2026 report: Xiaomi allegedly replayed its users' OpenClaw sessions through Claude for training data, and attackers prompt-injected OpenClaw deployments to steal API keys
- [[skill2env-capability-oriented-environment-synthesis-allspark-2026]] — Skill2Env: 3K public skills hardened into executable tasks; Terminal-Bench 2.1 44.9→58.4 (no distillation control)
