---
title: "pi-mono — Architecture & Implementation Notes"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - pi-mono.md
arxiv_id: null
venue: Technical write-up (latest-agents research corpus)
authors:
  - "(unattributed; internal technical doc)"
year: 2026
tags:
  - 2026
status: complete
importance: medium
introduces:
  - "[[pi-mono]]"
---

# pi-mono — Architecture & Implementation Notes

**Source**: `sources/pi-mono.md` (41KB technical document)
**Origin**: Architectural writeup of `earendil-works/pi-mono` (May 2026).

## Summary

[[pi-mono|pi]] (npm package `@earendil-works/pi-coding-agent`, binary `pi`) is a hand-crafted minimal terminal coding agent by **Mario Zechner ("badlogic")**. The project's defining posture is **editorial restraint** — it explicitly ships only:

**Four core tools**: `read`, `write`, `edit`, `bash`
**Three read-only tools**: `grep`, `find`, `ls`

And explicitly **refuses** to ship:
- Sub-agents
- Plan mode
- To-do tracking
- Background bash
- Permission popups
- MCP

Everything else (including the refused features) is expected to be built as a TypeScript Extension or installed as a Pi Package. pi-skills, a sibling repo, interoperates with Claude Code, Codex CLI, Amp, and Droid by reusing the [[skill-md-format|Agent Skills `SKILL.md` format]].

The implicit thesis: most "agent features" are agent-side wrappers that should be configurable per user, not baked into the core. The core should be small, predictable, and a stable substrate.

## Monorepo layout

Five npm-workspace packages plus example/extension workspaces:

| Package | Path | Purpose |
|---|---|---|
| `@earendil-works/pi-ai` | `packages/ai` | Provider-agnostic LLM streaming/completion client, model registry, OAuth helpers |
| `@earendil-works/pi-agent-core` | `packages/agent` | Stateful `Agent` class and low-level `agentLoop()` over `pi-ai` |
| `@earendil-works/pi-coding-agent` | `packages/coding-agent` | The `pi` CLI: tools, sessions, extensions, skills, TUI |
| `@earendil-works/pi-tui` | `packages/tui` | Terminal UI library with differential rendering |
| `@earendil-works/pi-web-ui` | `packages/web-ui` | Web components for provider dialogs and model discovery |

Dependencies flow: `pi-ai` → `pi-agent-core` → `pi-coding-agent`, with `pi-tui` consumed by `pi-coding-agent` and `pi-web-ui`. Versions are lockstep across packages.

## Key architectural facts

- **100% TypeScript, ESM, Node ≥20**.
- **Build with `@typescript/native-preview` (tsgo)**, `biome` for lint/format, `vitest` for tests.
- **Bun optional as compiled-binary target** but Node is canonical.
- **Provider-agnostic** via own `pi-ai` library — ~25 providers, 9 wire APIs.
- **Reused by OpenClaw** as its underlying runtime — pi-mono is one of the few agents that has been adopted as a substrate by another agent.
- **No memory layer in core**: AGENTS.md walk + JSONL session tree. Memory is an extension.
- **No permissions in core**: extensions can add their own.
- **Skills**: SKILL.md (Agent Skills v1) walked from `~/.pi/` and `~/.agents/`.

## Entities Mentioned

- [[pi-mono]] — the agent itself
- [[mario-zechner]] — author (badlogic)
- [[openclaw]] — uses pi-mono as runtime

## Concepts Discussed

- [[llm-agent]] — pi-mono is the minimalist instance
- [[tool-use]] — restricted core, extensible
- [[skill-md-format]] — SKILL.md as an interoperability format
- Editorial restraint as a design choice

## References

_Original source: `sources/pi-mono.md`_
_Also see [[cross-agent-comparison-2026|the cross-agent comparison]]._
