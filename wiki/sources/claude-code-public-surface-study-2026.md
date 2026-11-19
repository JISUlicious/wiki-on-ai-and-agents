---
title: "Claude Code public surface study (docs + public repo, 2026-11-18)"
type: source
created: 2026-11-18
updated: 2026-11-18
sources:
  - claude-code-public-surface-study-2026-11-18.md
year: 2026
url: https://code.claude.com/docs/en/how-claude-code-works
venue: "code.claude.com docs + github.com/anthropics/claude-code @ 52dd3a1"
authors:
  - Anthropic (documentation)
status: complete
importance: high
tags:
  - 2026
---

# Claude Code public surface study (2026-11-18)

## Summary

This page records what [[anthropic|Anthropic]] documents about [[claude-code|Claude Code]] as of 2026-11-18. It draws on 35 official doc pages (from a 205-page `llms.txt` index) and the public repo `anthropics/claude-code` at commit `52dd3a1` (2026-11-18; CHANGELOG top entry 2.1.327). **The implementation is closed source. Nothing was decompiled or run.** "[doc]" marks a claim from code.claude.com and "[repo]" a file seen in the public repo. Neither is verified against the binary. Raw notes are in `sources/claude-code-public-surface-study-2026-11-18.md`, and the fetched pages are in `sources/claude-code-docs-2026-11-18/`.

## Architecture

- [doc] One engine, many surfaces: CLI, Desktop, VS Code, JetBrains, Web (cloud VMs), Mobile, Slack/Claude Tag, GitHub Actions, GitLab CI. Execution can be Local, Cloud (Anthropic-managed or self-hosted) or Remote Control. "Scripting and the Agent SDK are CLI-only."
- [doc] The recommended install is a **native binary** that auto-updates (with ripgrep bundled). [repo] The README marks npm installation as deprecated but still carries a "Node.js 18+" badge.
- [doc] Providers: Anthropic API and subscriptions, Bedrock, Google Cloud Agent Platform (formerly Vertex), Microsoft Foundry, Claude Platform on AWS, and LLM gateways via `ANTHROPIC_BASE_URL`.
- [doc] The Agent SDK (Python/TS) runs "the same execution loop that powers Claude Code".
- [repo] `mods/` publishes the TypeScript source of six built-in [[claude-code-mods|mods]] (`sec-default`, `diff`, `telemetry`, `agents-md`, `privacy`, `hipaa-default`). Anthropic's plugins moved to `claude-plugins-official`, and `plugins/` now holds only a README.
- No mention of [[acp|ACP]] anywhere in the docs index.

## Agent loop

[doc] (`how-claude-code-works`, `agent-sdk/agent-loop`) The loop is gather context, act, verify, repeat. A turn is one model output plus the execution of its tool calls. The loop ends on a response with **no tool calls**. Optional caps are `maxTurns` and `maxBudgetUsd` (subagent spend counts toward the budget), giving result subtypes `error_max_turns` and `error_max_budget_usd`. **Read-only tools run concurrently; state-changing tools run sequentially.** Custom tools are sequential unless they carry `readOnlyHint`. Denials come back as tool results. Users can interrupt (`Esc`) or queue steering messages that are read mid-turn. A `Stop` hook can refuse to stop, and its input carries `stop_hook_active` so the hook can tell it is already continuing. Checkpoints snapshot files before each edit.

## Context & memory

- [doc] **CLAUDE.md** levels are managed, user, project (`./CLAUDE.md` or `./.claude/CLAUDE.md`) and local (`CLAUDE.local.md`). Ancestor files are concatenated root-first, and subdirectory files load lazily. There are path-scoped `.claude/rules/`, `@imports`, and a 4 MiB per-file cap. `AGENTS.md` is read only when no CLAUDE.md exists (implemented by the `agents-md` mod [repo]). See [[project-memory]].
- [doc] **Auto memory** is on by default and lives at `~/.claude/projects/<project>/memory/`, one per git repo. A `MEMORY.md` index (first 200 lines or 25KB loaded) points to on-demand topic files typed `user`/`feedback`/`project`/`reference`. **No background consolidator ("dream") is documented.**
- [doc] **Compaction** ([[context-compaction]]) summarizes the conversation. It triggers at the model's limit by default, or about 967K tokens on native-1M models; `/autocompact <tokens>` and `idleCompaction` adjust this, and `/compact [focus]` runs it by hand. Afterwards CLAUDE.md, auto memory, skill descriptions and the plan are re-injected from disk. Five recent files are re-read, and skill bodies return within a 5K-per-skill / 25K-total budget. A "thrashing" guard stops repeated auto-compaction.
- [doc] MCP tool schemas are deferred by default via [[tool-search]]. Skills load descriptions first and bodies on use.

## Tools, MCP & sub-agents

- [doc] `tools-reference` lists **45 current tools** (46 rows; `TaskOutput` is marked removed in 2.1.277). Many are conditional. `Task` was **renamed `Agent`** in 2.1.63. **Glob and Grep are absent by default on macOS/Linux/WSL**: search goes through Bash running embedded `bfs`/`ugrep`. `TodoWrite` is replaced by `TaskCreate/Get/List/Update`. New tools include `Workflow`, `Monitor`, `SendMessage`/`ListAgents`, `Artifact`, `RemoteTrigger`, `CronCreate` and `ToolSearch`.
- [doc] [[mcp|MCP]] supports HTTP (recommended), stdio and SSE (deprecated). Servers can push events as *channels* and request user input via elicitation.
- [doc] [[subagents|Sub-agents]]: the built-ins are Explore, Plan and general-purpose. Custom agents live in `.claude/agents/`. **Nesting is allowed up to 3 layers** (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`), with up to **20 concurrent**. Forks inherit the full conversation. Agent teams are experimental. [[dynamic-workflows]] run Claude-written orchestration scripts.
- [doc] [[agent-skills|Skills]] follow the agentskills.io open standard ([[skill-md-format]]), and custom commands have been merged into skills. Plugins bundle skills, agents, hooks, MCP and LSP servers, monitors and mods. There are four built-in output styles besides the default.

## Hooks & extension points

[doc] `hooks` documents **33 events** and **5 handler types**: `command`, `http`, `mcp_tool`, `prompt` (single-turn LLM) and `agent` (tool-using subagent; experimental). Matching hooks run in parallel.

| Phase | Events (bold = can block via exit 2) | Modify |
|---|---|---|
| Session | SessionStart, Setup, SessionEnd, InstructionsLoaded | SessionStart adds context |
| Prompt | **UserPromptSubmit**, **UserPromptExpansion**, MessageDisplay | context only; cannot rewrite prompt |
| Tool | **PreToolUse**, PermissionRequest (decision obj), PermissionDenied, PostToolUse, PostToolUseFailure, **PostToolBatch** | `updatedInput` (Pre/PermissionRequest), `updatedToolOutput` (Post), `retry` (Denied) |
| Stop | **Stop**, StopFailure, **SubagentStop**, **TeammateIdle** | force continue |
| Agents/tasks | SubagentStart, **TaskCreated**, **TaskCompleted** | rollback / prevent |
| Environment | **ConfigChange**, CwdChanged, DirectoryAdded, FileChanged, **WorktreeCreate**, **WorktreeRemove** | worktree hooks replace git |
| Context/model | **PreCompact**, PostCompact, **PreModelSwitch**, PostModelSwitch | — |
| MCP | **Elicitation**, **ElicitationResult** | decline |
| Other | Notification | — |

Hook `allow` never overrides deny or ask rules. [[claude-code-mods|Mods]] are a separate in-process layer.

## Guardrails: permissions & sandbox

| Layer | What is documented |
|---|---|
| Modes | `default` (UI name "Manual"), `acceptEdits`, `plan`, `auto`, `dontAsk` (CI: prompts become denials), `bypassPermissions` |
| Default mode | **`auto` for interactive terminal and VS Code sessions from v2.1.283**; `-p` and the SDK start in `default` |
| Auto classifier | rules first, then read-only and in-workspace edits auto-approved, then the rest goes to the classifier (default **Sonnet 5**); can run **server-side** inside the model request; no verdict means deny; also reviews `SendMessage` |
| Rules | `allow`/`ask`/`deny` with `Tool(specifier)`; order **deny, then ask, then allow**; deny holds even in bypass |
| Never auto-approved | ask rules, critical-path `rm`, `AskUserQuestion`/`requiresUserInteraction` MCP tools, org `ask` connectors |
| Sandbox | **off by default**; shell commands and their children only; macOS **Seatbelt**, Linux/WSL2 **bubblewrap + socat** (+ optional seccomp); native Windows unsandboxed; `@anthropic-ai/sandbox-runtime`; network via local proxy, `allowedDomains` start empty |
| Managed | settings precedence managed > CLI > local > project > user; `allowManagedPermissionRulesOnly`, `allowManagedHooksOnly`, `disableBypassPermissionsMode`, `disableAutoMode` |
| Devcontainer [repo] | `init-firewall.sh`: iptables default DROP, ipset allowlist (GitHub meta ranges, npm, api.anthropic.com, sentry, statsig, VS Code), self-test |

See also [[guardrailing]] and [[prompt-injection]].

## Goal command

[doc] `/goal <condition>` (up to 4,000 characters) is "a wrapper around a session-scoped prompt-based Stop hook". After each turn a small fast model judges the condition **from the transcript only** and returns not met, met or impossible. The loop stops after several turns without tool use. Unrecoverable errors clear the goal, and transient errors retry three times and then pause. Compare [[ralph-loop]] and [[goal-command-lineage-repos-2026]].

## Changes since the earlier wiki pages

- **Hooks: 33 events, not "21" or "pre/post/stop/error"**. The six new events since the May source list are UserPromptExpansion, MessageDisplay, PostToolBatch, DirectoryAdded, PreModelSwitch and PostModelSwitch. There are 5 handler types, including `mcp_tool` and `agent`.
- **Default permission mode is now auto** (v2.1.283), not Manual.
- **`Task` tool is `Agent`**. Glob and Grep are off by default on Unix. The current tool count is 45, not "~40" or "15+".
- **Subagents nest** (depth 3) and forks exist. Earlier pages treat sub-agents as flat.
- **No "auto-dream" consolidator** in the docs. Auto memory is a typed index plus topic files.
- **npm distribution deprecated**. A native binary is recommended.
- Plugin sources moved to `claude-plugins-official`, and built-in mods are published in `mods/`.

## Concepts & entities

[[claude-code]] · [[anthropic]] · [[claude]] · [[subagents]] · [[agent-skills]] · [[skill-md-format]] · [[mcp]] · [[tool-search]] · [[context-compaction]] · [[project-memory]] · [[claude-code-mods]] · [[dynamic-workflows]] · [[ralph-loop]] · [[guardrailing]] · [[prompt-injection]] · [[agentic-harness-engineering]] · [[acp]]

## References

- Docs: [how-claude-code-works](https://code.claude.com/docs/en/how-claude-code-works), [hooks](https://code.claude.com/docs/en/hooks), [tools-reference](https://code.claude.com/docs/en/tools-reference), [permission-modes](https://code.claude.com/docs/en/permission-modes), [permissions](https://code.claude.com/docs/en/permissions), [sandboxing](https://code.claude.com/docs/en/sandboxing), [memory](https://code.claude.com/docs/en/memory), [context-window](https://code.claude.com/docs/en/context-window), [sub-agents](https://code.claude.com/docs/en/sub-agents), [skills](https://code.claude.com/docs/en/skills), [goal](https://code.claude.com/docs/en/goal), [agent-sdk/agent-loop](https://code.claude.com/docs/en/agent-sdk/agent-loop)
- Repo: [anthropics/claude-code](https://github.com/anthropics/claude-code) @ `52dd3a1`: `README.md`, `CHANGELOG.md`, `mods/README.md`, `plugins/README.md`, `.devcontainer/init-firewall.sh`, `examples/`
- Earlier wiki sources: [[claude-code-agent-doc-2026]], [[agent-capabilities-check-2026-11]], [[claude-code-changelog-2026]], [[claude-code-mods-docs-2026]]
