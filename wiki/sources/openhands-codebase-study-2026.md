---
title: "OpenHands — Codebase Study (Agent Canvas + Software Agent SDK, 2026-11)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - openhands-codebase-study-2026-11-18.md
arxiv_id: null
venue: GitHub source code (OpenHands/OpenHands @ 37957771ba; OpenHands/software-agent-sdk @ f1eedd7919)
authors:
  - OpenHands contributors
year: 2026
url: https://github.com/OpenHands/OpenHands
tags:
  - 2026
status: complete
importance: medium
introduces:
  - "[[openhands]]"
---

# OpenHands — Codebase Study (2026-11)

**Repos read**: `OpenHands/OpenHands` @ `37957771ba` (Agent Canvas 1.41.0) and `OpenHands/software-agent-sdk` @ `f1eedd7919` (SDK/agent-server 1.62.0), both with commit date 2026-11-18. Both were shallow-cloned and read only. Nothing was built, installed or run. Raw notes: `sources/openhands-codebase-study-2026-11-18.md`.

## Summary

[[openhands|OpenHands]] (formerly OpenDevin) is now split across repos. The main `OpenHands/OpenHands` repo is no longer the Python agent. It is **Agent Canvas**, a React/TypeScript control center (`@openhands/agent-canvas`, plus an Electron build) that connects to one or more **Agent Servers** and can also run third-party agents over [[acp]]. The agent loop, tools, hooks, security and sandboxes live in the **Software Agent SDK**, a Python uv workspace with four packages: `openhands-sdk`, `openhands-tools`, `openhands-workspace` and `openhands-agent-server`. The repo's own `AGENTS.md` sets the dependency direction as Agent Server contract, then TypeScript client, then Canvas.

## Architecture

| Layer | Where | Role |
|---|---|---|
| Agent Canvas (TS/React, Vite, Zustand) | OH `src/`, `bin/agent-canvas.mjs`, `electron/` | UI, backend switching, automations UI; launches agent-server + automation via `uvx` |
| Agent Server (FastAPI + WebSocket) | SDK `openhands-agent-server/.../api.py` | ~30 routers (conversations, events, bash, git, files, VS Code, skills, sub-agents, plugins, hooks, MCP, ACP, settings); OpenAI-compatible gateway (`openai/`) |
| SDK core | SDK `openhands-sdk/openhands/sdk/` | `Agent`, `LocalConversation`/`RemoteConversation`, events, condenser, hooks, security, skills, plugins, MCP, LLM via LiteLLM |
| Tools | SDK `openhands-tools/` | terminal (tmux), file_editor, task_tracker, browser (14 functions), apply_patch, glob/grep, task/workflow sub-agents, presets |
| Workspaces | SDK `openhands-workspace/` | Docker, Apptainer, remote API, OpenHands Cloud, Kubernetes `agent_sandbox` |

`Conversation(...)` returns a `RemoteConversation` when the workspace is a `RemoteWorkspace`, and otherwise an in-process `LocalConversation`. A `DockerWorkspace` starts `ghcr.io/openhands/agent-server` in a container, and that server runs the local loop inside it.

## Agent loop

- **Outer loop**: `LocalConversation._run()` (`conversation/impl/local_conversation.py`) is a `while True` loop under a FIFO state lock. It runs over a status machine (IDLE, RUNNING, PAUSED, WAITING_FOR_CONFIRMATION, FINISHED, ERROR, STUCK). On FINISHED it runs **Stop hooks**. If a hook denies, its feedback is injected and the loop continues. The order of checks is stuck/nudge, then `agent.step()`, then the budget cap (`max_budget_per_run`), then `max_iteration_per_run` (default **500**).
- **Step**: `Agent._step()` (`agent/agent.py`):
  - It first executes any pending (already confirmed) actions.
  - It then asks the condenser for messages. A `Condensation` result is a step of its own.
  - It calls `llm.generate(..., add_security_risk_prediction=True)`.
  - It dispatches TOOL_CALLS, CONTENT or EMPTY responses (`response_dispatch.py`).
  - Errors become in-band corrections. Malformed function calls are fed back as user messages, content-filter blocks get a "rephrase" nudge, and context-window or malformed-history errors trigger a `CondensationRequest`.
- **Tool batch**: `_ActionBatch` cuts the batch at `finish`, turns hook-blocked actions into rejections, and runs the rest via `ParallelToolExecutor`. `tool_concurrency_limit` defaults to **1**. Above 1, tools run in parallel under per-resource locks (`declared_resources()`).
- **Stuck detection** (`conversation/stuck_detector.py`) covers 5 patterns: repeated action+observation (4), repeated action+error (3, with one nudge first), monologue, alternating A/O, and context-window-error loop.
- **LLM retries**: 5, exponential backoff (`llm/llm.py`).
- **Critic**: an optional critic can re-open a finished task when its score is below threshold (`critic_mixin.py`, iterative refinement).
- **`/goal`** (`conversation/goal/`): `GoalController` plus a judge LLM re-prompt the agent until the judge says done, up to a cap of 10 rounds. It is exposed as `POST /{conversation_id}/goal`.

## Context & memory

- **System prompt**: a `PromptRegistry` of ~19 named static sections (Soul, Role, Persona, Security, SecurityRiskAssessment, ToolGuidance, ModelSpecific…) and 6 dynamic ones (DateTime, RepoContext, MemoryContext, AvailableSkills, CustomSuffix, CustomSecrets). They are emitted as a `(static, dynamic)` pair for prompt caching. `persona` swaps only the persona/workflow sections.
- **Repo instructions**: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` and `.cursorrules` load as always-on "third-party skills". A nested `server/AGENTS.md` is directory-scoped (`skills/skill.py`).
- **Skills** (formerly *microagents*) load from `.agents/skills/`, `.openhands/skills/` and the deprecated `.openhands/microagents/`, at both project and user level.
  - Triggers are keyword, task, or path. Path-triggered "rules" are appended to the file observation that touched them.
  - `SKILL.md`-format skills are listed in the prompt and invoked via the auto-attached `invoke_skill` tool ([[agent-skills]], [[skill-md-format]]).
- **Memory**: a two-tier `MEMORY.md` index (`~/.openhands/memory/` and `<workspace>/.openhands/memory/`), capped at 6,000 chars and truncated from the top. Daily logs are read on demand ([[project-memory]]).
- **Condensers**: only `LLMSummarizingCondenser`, `NoOpCondenser` and `PipelineCondenser` remain. The default is a summarizer (`max_size=80`, `keep_first=4`) that sub-agents also use. It triggers on event count, a token cap, or a context-window error ([[context-compaction]]). The legacy monorepo's larger condenser zoo is not in the SDK.

## Tools, MCP & sub-agents

- **Default toolset** (`tool/defaults.py`): `terminal`, `file_editor`, `task_tracker`, `switch_llm`, plus built-in `finish`, which makes **5**. When a browser is available, `browser_tool_set` adds **14** functions, for **19** in total. `invoke_skill` is added when skills exist. Opt-in built-ins are `think`, `vision_inspect` and `classify_and_switch_llm`.
- **Presets**: a GPT-5 preset (apply_patch), a Gemini preset (read/write/edit/list), and a planning preset (glob, grep, `PLAN.md` editor).
- **CodeAct naming**: the settings still default `agent="CodeActAgent"`, but actions are native function calls to terminal/editor tools rather than executable Python code ([[codeact]]).
- **[[mcp]]**: `fastmcp` client with stdio/HTTP transports and OAuth. Servers can be added at runtime (`add_mcp_servers`), and plugins ship `.mcp.json`.
- **[[acp]]**: `ACPAgent` (`agent/acp_agent.py`, ~5k lines) runs an external ACP agent as a subprocess. Registered providers are claude-code, codex, gemini-cli, kimi-code, pi and opencode. OpenHands acts as an ACP *host*. I did not find it acting as an ACP server in these repos.
- **[[subagents]]**: `task(description, prompt, subagent_type)` over Markdown/YAML agent definitions. The built-ins are general-purpose, bash-runner, code-explorer and web-researcher.
  - `start_child_conversation` launches a separate server conversation.
  - `WorkflowToolSet` lets the agent write an `async def main(wf)` Python script that fans out sub-agent tasks (concurrency 8, max 64), a form of [[dynamic-workflows]].

## Hooks & extension points

| Hook / point | Fires | Can block? | Can modify? |
|---|---|---|---|
| `PreToolUse` | on each `ActionEvent`, before execution | yes (exit 2 / deny → rejection observation) | no arg rewrite found |
| `PostToolUse` | on each `ObservationEvent` | no | no (logged only) |
| `UserPromptSubmit` | on user `MessageEvent` | yes (turn ends FINISHED) | adds context |
| `SessionStart` | conversation init / plugin load | no | context injected after system prompt |
| `SessionEnd` | conversation close | no | no |
| `Stop` | when status hits FINISHED | yes (deny keeps agent running + feedback) | feedback text |
| Callbacks / token callbacks | every event / token | no | no |
| Plugins | load time or `load_plugin()` | — | bundle skills, hooks, MCP, agents, commands |

There are **6 hook events** (`hooks/types.py`) and three handler types: `command` (JSON on stdin), `prompt` (one LLM verdict) and `agent` (an agent with tools). Exit codes follow what the code calls "Claude Code's hook contract" (0 ok, 2 block). Config comes from `.openhands/hooks.json`, then `~/.openhands/hooks.json`, then plugin `hooks/hooks.json`. Plugins follow the Claude Code layout (`.claude-plugin/plugin.json`). See [[hooks-in-modern-agents]].

## Guardrails: permissions & sandbox

| Mechanism | Detail (file) |
|---|---|
| Confirmation policy | `AlwaysConfirm` / `NeverConfirm` / `ConfirmRisky(threshold=HIGH, confirm_unknown=True)` → `WAITING_FOR_CONFIRMATION` (`security/confirmation_policy.py`) |
| Self-reported risk | Every tool schema gets a `security_risk` field. `LLMSecurityAnalyzer` trusts the acting model's own label |
| Independent analyzers | `ToolShieldLLMSecurityAnalyzer` (separate guardrail LLM), `PatternSecurityAnalyzer` + `PolicyRailSecurityAnalyzer` (tree-sitter shell AST; fetch-to-exec, raw-disk, catastrophic-delete rails), `GraySwanAnalyzer`, `EnsembleSecurityAnalyzer` (max risk) |
| Hooks | PreToolUse deny, Stop deny |
| Sandbox | Workspace choice: local (none), Docker, Apptainer, remote, Cloud, K8s agent-sandbox (gVisor/Kata optional) |
| Network | No egress allowlist in SDK code; Docker `--network` option; K8s template default allows public egress |
| Secrets | `SecretRegistry` masks values in observations and security events; the prompt lists secret names only |
| Limits | 500 iterations/run, cost budget per run, stuck detector, goal cap 10, 5 LLM retries |

([[guardrailing]], [[prompt-injection]])

## Distinctive points

1. **Frontend/engine split**: the flagship repo is a UI that also hosts *other* vendors' agents via ACP. The engine is a separately versioned SDK with a REST/WebSocket contract and a TypeScript client.
2. **Risk-labelled tool calls**: the acting model labels every action LOW/MEDIUM/HIGH, and pluggable analyzers can override that label. Confirmation is a policy over risk, not over tool names.
3. **Event-sourced, forkable conversations** (`fork`, `navigate_to`, `rerun_actions`). Condensation is itself an event.
4. **Strong Claude Code convergence**: same hook names and exit codes, plugin layout, `task` sub-agent tool, `MEMORY.md`, `/goal` with a judge, and workflow scripts.
5. **Sandbox-as-workspace**: isolation is chosen by swapping the workspace class, with the same agent-server image used everywhere.

## Concepts & entities

[[openhands]] · [[openhands-wang-2024]] · [[codeact]] · [[acp]] · [[mcp]] · [[subagents]] · [[dynamic-workflows]] · [[agent-skills]] · [[skill-md-format]] · [[project-memory]] · [[context-compaction]] · [[guardrailing]] · [[prompt-injection]] · [[llm-as-a-judge]] · [[hooks-in-modern-agents]] · [[claude-code]] · [[codex-cli]] · [[gemini-cli]] · [[opencode]] · [[pi-mono]] · [[swe-bench]]

## References

- https://github.com/OpenHands/OpenHands (commit `37957771ba`)
- https://github.com/OpenHands/software-agent-sdk (commit `f1eedd7919`)
- `sources/openhands-codebase-study-2026-11-18.md`
- Not read: `OpenHands/OpenHands-CLI`, `OpenHands/extensions`, `OpenHands/automation`.
