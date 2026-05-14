---
title: "Claude Code — Architecture & Implementation Notes"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - claude-code.md
arxiv_id: null
venue: Technical write-up (latest-agents research corpus)
authors:
  - "(unattributed; internal technical doc)"
year: 2026
tags:
  - 2026
status: complete
importance: high
introduces:
  - "[[claude-code]]"
---

# Claude Code — Architecture & Implementation Notes

**Source**: `sources/claude-code.md` (50KB technical document)
**Origin**: Snapshot derived from an [[anthropic|Anthropic]] `@anthropic/claude-code` npm sourcemap leak (March 2026), curated into a structured architecture writeup.

## Summary

This is a deep architectural writeup of **[[claude-code|Claude Code]]**, [[anthropic|Anthropic]]'s official terminal-based coding agent. Far from being "a CLI that hits the Claude API," it is a local agent operating system:

- ~5,000-line REPL orchestrator (`main.tsx`)
- ~1,700-line turn state machine (`query.ts`)
- Unified tool contract with ~40 built-in tools
- First-class MCP / plugin / skill extensibility
- Multi-agent subsystem with worktree and remote isolation
- File-based memory with a background "auto-dream" consolidator
- Layered permission / sandbox / hooks stack that separates *whether a call is allowed* from *what runtime capabilities it has*

The document covers two entry surfaces (`cli.tsx` minimal launcher and `main.tsx` real composition root), the runtime spine, tool architecture, sub-agent orchestration, MCP integration, the permission system, memory subsystem, sandbox stack, and the hooks framework — all with file paths and line counts.

## Key architectural facts

- **Built with React/Ink TUI**.
- **Tool system**: ~40 built-ins including Bash, Read, Edit, Write, Glob, Grep, Task (sub-agent dispatch), MCP, WebFetch, WebSearch, etc. Tools are uniform Zod-schema'd contracts.
- **Sub-agents**: first-class via the `Task` tool. 6+ built-in subagent types (Explore, Plan, code-reviewer, etc.). Subagents can run in worktree-isolated or remote-isolated modes.
- **Memory**: file-based at `~/.claude/projects/{slug}/memory/` with an auto-consolidator ("auto-dream") that periodically distills accumulated memory.
- **Permissions**: rules + classifier + plan-mode reminder + hooks. Permission system explicitly separates *allowed* from *capable*.
- **Sandbox**: OS-level via `@anthropic-ai/sandbox-runtime` adapter (FS/network rules).
- **MCP**: first-class, in-tree. Deferred-tool loading via `ToolSearch` is part of the standard kit.
- **Hooks**: pre-tool, post-tool, on-stop, on-error. Configurable in `settings.json`.
- **CLAUDE.md**: nested in project tree; auto-loaded at startup. Holds domain-specific schema.

## Entities Mentioned

- [[claude-code]] — the agent itself
- [[anthropic]] — origin org
- [[claude]] — the underlying model family

## Concepts Discussed

- [[llm-agent]] — Claude Code is a canonical instance
- [[tool-use]] — extensive built-in toolbox
- Sub-agent orchestration
- File-based memory / context engineering
- Sandboxing for agent capability containment

## References

_Original source: `sources/claude-code.md`_
_Also see [[cross-agent-comparison-2026|the cross-agent comparison]] which contrasts Claude Code with opencode, pi-mono, hermes-agent, and openclaw._
