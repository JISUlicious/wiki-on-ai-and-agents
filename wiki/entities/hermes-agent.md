---
title: Hermes Agent
type: entity
created: 2026-05-14
updated: 2026-11-18
sources:
  - hermes-agent.md
  - comparison.md
  - hermes-agent-codebase-study-2026-11-18.md
family: agent-product
parameters: "n/a (provider-agnostic, OpenAI-format chat completions)"
release_date: 2026-02-25
status: complete
importance: medium
tags:
  - product
---

# Hermes Agent

> [!note] Dates
> The repository began 2025-07-22. The 2026-02-25 launch date comes from secondary sources only. See [[agent-products-repo-history-2026]].

Hermes Agent (`NousResearch/hermes-agent`) is **[[nous-research|Nous Research]]**'s open-source, self-improving AI agent. Python-based. Runs as either an interactive terminal (`hermes`) or a persistent multi-platform messaging daemon (`hermes gateway`). Tagline from its README:

> "The self-improving AI agent built by Nous Research. It's the only agent with a built-in learning loop — it creates skills from experience, improves them during use, nudges itself to persist knowledge, searches its own past conversations, and builds a deepening model of who you are across sessions."

Provider-agnostic. As of commit `0e8e421640` (2026-11-18), providers are 39 plugins in `plugins/model-providers/` (OpenRouter, Nous Portal, [[anthropic|Anthropic]], NVIDIA, [[huggingface|Hugging Face]], Ollama Cloud, LM Studio, Bedrock, Vertex, xAI, and others). It is no longer chat-completions only: `api_mode` also selects native Anthropic, Bedrock, Gemini and Vertex adapters, Codex Responses, or an opt-in Codex app-server subprocess ([[hermes-agent-codebase-study-2026]]).

## Distinguishing feature: self-modifying runtime

What sets Hermes apart from the other documented 2026 agents: **the runtime *and* the persistence layer (skills, memory, sessions, cron jobs) are themselves edited by the agent during normal use**. "Growing" is treated as a first-class system concern, not a UX flourish.

Concretely:
- The agent can create new skills from experience and modify them during use.
- Background "nudging" mechanism for persisting knowledge.
- 4-layer memory: frozen MEMORY.md/USER.md snapshot + provider plugin + FTS5 session search + auto-archive curator.

## Codebase scale

> [!warning] Contradiction
> The May 2026 development guide ([[hermes-agent-doc-2026]]) described `run_agent.py` as the core loop at ~15.7k LOC, `cli.py` at ~13.5k LOC, and `hermes_state.py` as the SQLite + FTS5 SessionDB. At commit `0e8e421640` ([[hermes-agent-codebase-study-2026]]), `run_agent.py` is a 1,621-line `AIAgent` facade. The turn loop is `agent/conversation_loop.py` plus 33 `agent/turn_*.py` phase modules, and `cli.py` is 1,797 lines. `hermes_state.py` is a deprecation shim, to be removed in v0.27.0; storage lives in `persistence/`.

- About 12.7k Python files, of which about 9.1k are tests. The largest packages are `hermes_cli/` (~327k lines), `agent/` (~164k), `gateway/` (~140k) and `tools/` (~136k).
- Repo rules cap new functions at 300 lines and files at 2,000. Two design invariants come first: prompt caching is sacred (compression is the only allowed cache break), and the core is a narrow waist, so new capability arrives as skills, plugins or MCP servers.

## Architecture

- **Multi-surface**: classic CLI, Ink TUI, Electron desktop app, web dashboard, a messaging gateway with 26 platform plugins plus an OpenAI-compatible API server, and an ACP adapter for editors. The TUI, desktop app and dashboard attach over WebSocket to a per-profile gateway-owner process.
- **Agent loop**: synchronous, phase-split loop. The iteration cap is unlimited by default (`max_iterations = sys.maxsize`; `agent.max_turns: null`). Tool batches are split into parallel and sequential segments by read-only status and path overlap, with up to 8 workers. A verification nudge stops the agent from finishing right after a code edit with no fresh check.
- **Sandboxing**: 6 in-tree terminal backends: local, Docker, SSH, Modal, Daytona and Vercel Sandbox. Singularity is now a catalog plugin, and plugins can register more.

> [!warning] Contradiction
> This page previously said, from the May source ([[hermes-agent-doc-2026]]): "7 backends — local, Docker, SSH, Singularity, Modal, Daytona, Vercel Sandbox". At `0e8e421640`, `tools/terminal_tool_backends.py::_ENV_BUILDERS` lists 6. Singularity is `plugin-catalog/singularity.yaml` ([[hermes-agent-codebase-study-2026]]).

- **Sub-agents**: `delegate_task` with leaf/orchestrator roles. Default depth is 1 (`delegation.max_spawn_depth` raises it), with up to 10 concurrent children and a 250-iteration cap per child. Children can't delegate, ask the user, write memory, send messages or schedule cron. A kanban dispatcher also spawns worker agents.
- **Permissions**: `approvals.mode` can be `manual` (default), `smart` (an auxiliary-LLM guardian) or `off`, plus a per-session `/yolo`. Hardline patterns can't be approved even under YOLO. Answers can apply once, for the session, or always. ACP clients get `request_permission`.
- **Hooks**: 48 plugin hooks in `VALID_HOOKS` (46 on 2026-11-02), four middleware kinds, and Claude Code-style shell hooks where exit 2 blocks `pre_tool_call`. See [[hooks-in-modern-agents]].
- **MCP**: client over stdio, SSE and streamable HTTP, with OAuth, elicitation and sampling; 66 curated configs in `optional-mcps/`. MCP and plugin tools sit behind `tool_search` progressive disclosure. `hermes mcp serve` moved out of core into the `mcp-serve` plugin.
- **Skills**: SKILL.md (Anthropic-inspired, agentskills.io compatible): 44 bundled and 141 optional. The agent edits skills via `skill_manage`, a curator maintains them, and `skills_guard` scans external ones.
- **Tools**: 68 built-ins registered statically in `tools/`; the default core list (`_HERMES_CORE_TOOLS`) has 41. These include memory, session_search, terminal, delegate_task, execute_code, browser tools and 14 kanban tools.
- **Memory**: the four layers still exist. In-tree memory-provider plugins are now only byterover and retaindb; honcho, mem0 and others are catalog plugins.

**Goal and trigger features:** `/goal` (2026-05-01; `hermes_cli/goals.py`, "the Ralph loop for Hermes") runs shell quality gates and then an auxiliary judge. The default is 20 turns, and the judge fails open. `/heartbeat`, `/loop` and cron start work on a schedule. See [[agent-goal-loops-extension-docs-2026]].

## References

- [[hermes-agent-codebase-study-2026]] — source read at `0e8e421640` (2026-11-18)
- [[hermes-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]

## Related 2026 sources

- [[ollama]] — Ollama: local model runner on llama.cpp/GGUF; native /v1/systemone decision models since v0.35.0
