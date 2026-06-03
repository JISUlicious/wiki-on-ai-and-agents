---
title: Claude Code
type: entity
created: 2026-05-14
updated: 2026-06-03
sources:
  - claude-code.md
  - comparison.md
  - claude-code-dynamic-workflows-2026.md
  - claude-code-dynamic-workflows-harness-2026.md
  - claude-effort-and-ultracode-docs-2026.md
family: agent-product
parameters: "n/a (powered by Claude)"
release_date: 2024-02-29
status: complete
importance: high
tags:
  - product
---

# Claude Code

Claude Code is [[anthropic|Anthropic]]'s official terminal-based coding agent. Distributed as the `@anthropic/claude-code` npm package. Powered by [[claude|Claude]] models (Opus / Sonnet / Haiku). The CLI maintaining this very wiki is Claude Code.

## Architecture (per the 2026 sourcemap-snapshot writeup)

- **TypeScript / React-Ink TUI**.
- **~5,000-line REPL orchestrator** (`main.tsx`), **~1,700-line turn state machine** (`query.ts`).
- **~40 built-in tools** with a unified Zod-schema contract: Bash, Read, Edit, Write, Glob, Grep, Task (sub-agent dispatch), MCP, WebFetch, WebSearch, NotebookEdit, AskUserQuestion, ExitPlanMode, and more.
- **Sub-agents are first-class** via the `Task` tool; built-in subagent types include Explore, Plan, code-reviewer, claude-code-guide, etc. Subagents can run in worktree-isolated or remote-isolated modes.
- **Memory subsystem**: per-project file-based memory at `~/.claude/projects/{slug}/memory/`, with a background "auto-dream" consolidator that periodically distills memory into compact reusable forms.
- **Permissions system**: layered. Rules + classifier + plan-mode + hooks. Separates *whether* a call is allowed from *what runtime capabilities* it has.
- **Sandbox**: OS-level via `@anthropic-ai/sandbox-runtime` adapter (FS/network rules).
- **MCP**: first-class. Tools can be deferred and loaded on demand via `ToolSearch`.
- **Hooks**: pre-tool, post-tool, on-stop, on-error. Configurable in `settings.json`.
- **CLAUDE.md**: nested in project tree, auto-loaded at startup, holds domain-specific schema (the wiki you are reading lives in such a CLAUDE.md ecosystem).

## Dynamic workflows (2026)

As of May–June 2026, Claude Code can **write and orchestrate its own multi-agent harness on the fly** via [[dynamic-workflows]]: Claude authors a JavaScript orchestration script that spawns tens-to-hundreds of parallel subagents (reusing the `Task`-tool subagent + worktree machinery above), executed by a background runtime while the session stays responsive. Enabled by [[claude-opus-4-8|Claude Opus 4.8]]; surfaced via the [[ultracode]] setting in the `/effort` menu (which pairs `xhigh` [[reasoning-effort|effort]] with standing workflow-launch permission) and composable with `/loop` and `/goal`. (Distinct from the unofficial community [[ultrawork]] trend.) See the [[claude-code-dynamic-workflows-2026|announcement]] and [[claude-code-dynamic-workflows-harness-2026|engineering deep-dive]]. This makes the self-authored harness a first-class Claude Code capability — the [[code-as-harness]] thesis applied to Claude Code itself.

## In the cross-agent landscape

- Distinguished by **scale** (~40 tools, first-class sub-agents, layered permissions, OS-level sandbox) — most other documented agents are simpler.
- **Compatible with the broader SKILL.md ecosystem**: pi-mono, opencode, Hermes, and OpenClaw all walk the same skill directories that Claude Code does, treating SKILL.md as a de-facto interop format.
- Source code was leaked via npm sourcemap in March 2026 and was mirrored locally for the writeup.

## References

- [[claude-code-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]
- [[claude-code-dynamic-workflows-2026]] — dynamic workflows announcement
- [[claude-code-dynamic-workflows-harness-2026]] — dynamic workflows engineering deep-dive
