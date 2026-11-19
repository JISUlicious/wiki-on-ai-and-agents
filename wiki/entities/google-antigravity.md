---
title: Google Antigravity
type: entity
created: 2026-11-18
updated: 2026-11-18
sources:
  - antigravity-public-surface-study-2026-11-18.md
  - antigravity-docs-2026-11-18/
status: complete
importance: medium
tags:
  - product
---

# Google Antigravity

[[google|Google]]'s agentic coding platform. It started as an agent-first IDE, which the caller dates to November 2025; the fetched docs don't give a launch date. It is now a product line of five surfaces that share one closed agent engine.

The terminal surface has its own page, [[antigravity-cli]]. This page covers the platform, the App and the IDE. Everything below comes from public docs and repos, as summarised in [[antigravity-public-surface-study-2026]]. Nothing was verified in source, because the engine is not open source.

## Surfaces (documented)

| Surface | What it is |
|---|---|
| **Antigravity App** (v2.5+) | A desktop "command center" for launching and inspecting agents. The docs say it evolved from the **Agent Manager**, and the CLI README calls it "Antigravity 2.0". |
| **Antigravity IDE** (v2.6+) | The standalone editor, with an agent side panel, inline diff review and an IDE browser. It is not supported for enterprise. |
| **IDE extensions** | VS Code, JetBrains, Visual Studio, [[zed\|Zed]] and Xcode (GA), plus Eclipse (preview). |
| **[[antigravity-cli\|Antigravity CLI]]** (`agy`) | A TUI and headless client that can also act as an [[acp\|ACP]] server. |
| **Antigravity SDK** (Python) | An Apache-2.0 wrapper that drives a bundled compiled runtime binary (repo). |

The CLI README says the CLI and the App run on a "Shared Core Agent Engine" with synced settings and permissions (repo). So the IDE and CLI are separate products on one harness, which is why they get two pages.

## What it does (documented)

- **Models**: Gemini 4 Carbon, Argon and Hydrogen, Gemini 3.x Flash, Claude Opus and Sonnet 5.5, and GPT-OSS-120b. There is also an **Orchestrator** entry in which a planner model hands implementation to Hydrogen within one turn.
- **Artifacts and review**: Planning mode produces implementation plans, diffs, walkthroughs and browser recordings for inline-comment review. The Plan Review Policy is *Always Ask* or *Always Proceed*.
- **Context**:
  - Rules come from both `AGENTS.md` and `GEMINI.md`, subject to a 24 KB per-file limit and a 20k-token rule budget.
  - [[agent-skills|Skills]] use `SKILL.md`.
  - [[mcp|MCP]] is lazy-loaded through on-disk schemas and `call_mcp_tool`.
  - There is no automatic memory, but `/learn` proposes rules or skills.
- **Multi-agent**:
  - [[subagents]] (`research`, `browser`, `self` and custom ones, optionally in a git worktree).
  - `/swarm`, `/teamwork` and `/run-workflow` (Python-scripted fan-out, like [[dynamic-workflows]]).
- **Browser subagent**: the App has a built-in browser. The CLI and IDE use Chrome DevTools MCP against the user's Chrome, version 144 or later. It is governed by Browser Actuation Rules.
- **Hooks**: six command hooks, `SessionStart`, `PreToolUse`, `PostToolUse`, `PreInvocation`, `PostInvocation` and `Stop`, shared across the App, CLI and IDE. See [[hooks-in-modern-agents]].
- **Guardrails**:
  - An `action(target)` permission engine evaluated Deny > Ask > Allow.
  - Presets: Request Review, Default, Auto and Turbo.
  - An OS terminal sandbox using Linux namespaces or macOS Seatbelt, with no network by default.
  - Permission grants compile into the sandbox's mounts and its network allowlist. See [[guardrailing]].
- **Long-running work**: `/goal` uses a self-report stop hook with no cap. `/schedule` sets conversation-scoped timers and cron jobs. `/automation` and sidecars handle unattended jobs, and the docs list them as App-only.
- **Deprecated**: IDE "Workflows" are being replaced by skills.

## Relationship to other Google agents

- [[gemini-cli]] remains for Code Assist Standard and Enterprise. Consumer tiers moved to [[antigravity-cli]] on 2026-06-18.
- Plugins can import Gemini CLI extensions and [[claude-code|Claude Code]] plugins.
- [[google-adk]] is a separate agent framework. None of the 54 fetched doc pages mention ADK.

## References

- [[antigravity-public-surface-study-2026]]
- [[antigravity-cli-docs-2026-11]]
- https://antigravity.google/docs/overview
