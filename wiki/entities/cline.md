---
title: Cline
type: entity
created: 2026-11-18
updated: 2026-11-18
sources:
  - cline-codebase-study-2026-11-18.md
family: agent-product
parameters: "n/a (provider-agnostic)"
url: https://github.com/cline/cline
status: complete
importance: medium
tags:
  - product
---

# Cline

Cline is an open-source (Apache-2.0) autonomous coding agent from Cline Bot Inc. It is best known as a VS Code extension, and now also ships as a TUI CLI, a desktop app and chat-platform bots. It began as **"Claude Dev"**. The extension's package name is still `claude-dev` (publisher `saoudrizwan`), and it was renamed to Cline at v2.0.0. That product is unrelated to this wiki's [[claude-dev]] page, which covers Anthropic's claude.dev blog. Forks such as Roo Code and Kilo Code descend from Cline. That lineage is background knowledge only; this study did not examine them.

Code-level details: [[cline-codebase-study-2026]] (commit `eff43e19b2`, 2026-11-17).

## Lineage

Cline started as a single extension around a `Task` class. Its `recursivelyMakeClineRequests` loop used XML-style tools, one tool per message by default, and `attempt_completion` to stop. By late 2026 the agent had been rebuilt as a layered **Cline SDK** in a Bun/TypeScript monorepo, and the old extension survives on a `legacy-extension` branch.

The marketplace VSIX ships both bundles. A small loader picks the SDK bundle or the legacy bundle per window by feature-flag cohort and falls back to legacy on crash.

## Code-level overview (main, 2026-11)

- **Language/runtime**: TypeScript, Bun toolchain, Node ≥22. Providers go through the Vercel AI SDK: 53 hand-written provider IDs plus 218 generated from models.dev. The default `cline` gateway provider needs sign-in or `CLINE_API_KEY`.
- **Layers**: `@cline/shared → @cline/llms → @cline/agents` (stateless `AgentRuntime` loop) `→ @cline/core` (sessions, tools, plugins, compaction, hub daemon, cloud).
- **Process model**: in-process, or through a detached **hub daemon** shared by every Cline client on the machine. Sessions survive client detach. Cloud sandbox sessions are a third option.
- **Loop**:
  - Native tool calls, with **parallel batching required** by the prompt.
  - A turn without tool calls completes the run. `submit_and_exit` exists only in yolo mode.
  - Recovery: retries, output-limit recovery, a mistake limit (6 in core, 3 in the CLI), loop detection (3/5).
- **Modes**: Plan/Act/YOLO, plus the CLI's `zen` background mode. Plan mode keeps the shell but blacklists file-mutating commands through a `beforeTool` hook.
- **Tools**: 9 default SDK tools, plus `spawn_agent`, 18 `team_*` tools, `tasks` (Todos and cron schedules), [[mcp]] tools (with a marketplace) and `skills`. GPT/Codex models get `apply_patch` instead of `editor`.
- **Context**:
  - `AGENTS.md`, `.clinerules`/`.cline/rules`, workflows as slash commands.
  - [[agent-skills]], which also reads `.claude/skills`.
  - [[context-compaction]] at 90% of budget, agentic or basic.
  - Checkpoints as private git refs in the user's repo.
- **Hooks**: 7 in-process runtime hooks and 10 file-based script hooks (`TaskStart` … `PreToolUse`/`PostToolUse` … `SessionShutdown`). The VS Code SDK bundle wires only 6 of the file hooks. See [[hooks-in-modern-agents]].
- **Guardrails**: per-tool approval policies, VS Code auto-approve categories (reads, edits, web and MCP on by default; commands off), and YOLO. The CLI defaults one-shot prompts to yolo. There is **no OS sandbox** for commands.
- **Surfaces**: the VS Code extension (gRPC webview), a standalone host bridge that JetBrains uses, the CLI with [[acp]] mode, Slack/Discord/Telegram/WhatsApp/Google Chat/Linear connectors, and a Tauri desktop example.

## Related

[[claude-code]] · [[cursor]] · [[opencode]] · [[pi-mono]] · [[hermes-agent]] · [[codex-cli]] · [[subagents]] · [[cross-agent-comparison-2026]]
