---
title: OpenHands
type: entity
created: 2026-11-18
updated: 2026-11-18
sources:
  - openhands-codebase-study-2026-11-18.md
  - openhands-wang-2024.md
status: complete
importance: medium
tags:
  - product
---

# OpenHands

OpenHands is an open-source (MIT) platform for software-development agents. It started in 2024 as **OpenDevin**, an open reproduction of [[devin|Devin]]. It was later renamed OpenHands, and its GitHub organization moved from `All-Hands-AI` to `OpenHands`. The research paper is [[openhands-wang-2024]] (ICLR 2025), with authors including [[xingyao-wang|Xingyao Wang]] and [[graham-neubig|Graham Neubig]]. Its original agent was [[codeact|CodeAct]], and it is a common harness for [[swe-bench]] runs. Researchers reuse its sandbox, tools and event stream as a shared reference harness, which makes it a widely cited instance of [[code-as-harness]] infrastructure. (This page absorbed the earlier draft concept page of the same name on 2026-11-18.)

## Code-level overview (2026-11)

From [[openhands-codebase-study-2026]] (repo commits `37957771ba` and `f1eedd7919`, read only, nothing run):

- **Repo split**:
  - `OpenHands/OpenHands` is now **Agent Canvas**, a React/TypeScript UI with an Electron build. It drives one or more Agent Servers and can run third-party agents over [[acp]] (Claude Code, Codex, Gemini CLI, Pi, OpenCode, Kimi).
  - The engine is **`OpenHands/software-agent-sdk`**, a Python package set: `openhands-sdk`, `openhands-tools`, `openhands-workspace` and `openhands-agent-server` (FastAPI + WebSocket, plus an OpenAI-compatible gateway).
  - The CLI lives in a separate repo, `OpenHands-CLI`.
- **Loop**: `LocalConversation._run()` runs a status machine and calls `Agent._step()`. Each step makes one LLM call and runs a tool batch.
  - Limits: 500 iterations per run, an optional cost budget, and a stuck detector with 5 patterns.
  - Stop hooks can keep the agent running after it finishes.
  - Tool calls run sequentially by default; parallel execution with per-resource locks is opt-in.
- **Context**:
  - The system prompt is built from sections split into static and dynamic tiers for caching.
  - `AGENTS.md`, `CLAUDE.md` and `GEMINI.md` load as always-on skills. Other skills (formerly *microagents*) use keyword, task or path triggers, and `SKILL.md` skills are invoked through `invoke_skill`.
  - Memory is a two-tier `MEMORY.md` index.
  - The default condenser is an LLM summarizer ([[context-compaction]]).
- **Tools**: the default set is terminal (tmux), file_editor, task_tracker, switch_llm and finish. A 14-function browser set is added when a browser is available.
  - Integrations: [[mcp]] via fastmcp, and the `task` tool for [[subagents]].
  - The agent can also write Python workflow scripts that fan out sub-agents ([[dynamic-workflows]]).
  - Goal mode: `/goal` uses a judge LLM.
- **Hooks**: 6 events: PreToolUse, PostToolUse, UserPromptSubmit, SessionStart, SessionEnd and Stop.
  - Handlers can be `command`, `prompt` or `agent`, and use Claude-Code-compatible exit codes.
  - Plugins use the Claude Code layout ([[hooks-in-modern-agents]]).
- **Guardrails**: the acting model labels every tool call with a risk level.
  - Security analyzers can override that label: LLM, a ToolShield guardrail LLM, tree-sitter shell policy rails, GraySwan, or an ensemble.
  - A confirmation policy (`ConfirmRisky` etc.) pauses risky calls for approval.
  - Isolation comes from choosing a workspace: local, Docker, Apptainer, remote, Cloud, or a Kubernetes agent-sandbox. There is no built-in egress allowlist ([[guardrailing]]).
  - Secret values are masked in observations.

## Related

[[claude-code]] · [[opencode]] · [[pi-mono]] · [[codex-cli]] · [[gemini-cli]] · [[cross-agent-comparison-2026]]

## References

- [[openhands-codebase-study-2026]]
- [[openhands-wang-2024]]
