---
title: pi (pi-mono coding agent)
type: entity
created: 2026-05-14
updated: 2026-05-14
sources:
  - pi-mono.md
  - comparison.md
family: agent-product
parameters: "n/a (provider-agnostic)"
release_date: 2024-12-01
status: complete
importance: medium
tags:
  - product
---

# pi (pi-mono coding agent)

pi (`@earendil-works/pi-coding-agent`, binary `pi`) is a hand-crafted minimal terminal coding agent by **Mario Zechner ("badlogic")**, distributed from the `earendil-works/pi-mono` monorepo. Among the documented 2026 agents, pi is the **minimalist** — it ships only what's strictly needed and pushes everything else into extensions.

## What pi ships

**Core tools (4)**: `read`, `write`, `edit`, `bash`
**Read-only tools (3)**: `grep`, `find`, `ls`

Total: **7 built-in tools**. Compare to Claude Code's ~40.

## What pi explicitly refuses (in core)

- Sub-agents
- Plan mode
- To-do tracking
- Background bash
- Permission popups
- MCP

Everything refused is expected to be implemented as a **TypeScript Extension** or installed as a **Pi Package**. The sibling repo **pi-skills** packages skills in the `SKILL.md` format, explicitly designed to interoperate with Claude Code, Codex CLI, Amp, and Droid.

## Architecture

Five-package npm workspace:

| Package | Purpose |
|---|---|
| `@earendil-works/pi-ai` | Provider-agnostic LLM client, model registry, OAuth helpers |
| `@earendil-works/pi-agent-core` | Stateful `Agent` class, low-level `agentLoop()` |
| `@earendil-works/pi-coding-agent` | The `pi` CLI |
| `@earendil-works/pi-tui` | Terminal UI library with differential rendering |
| `@earendil-works/pi-web-ui` | Web components for provider dialogs |

- **100% TypeScript, ESM, Node ≥20**.
- **Build**: `@typescript/native-preview` (tsgo), `biome`, `vitest`.
- **Bun optional** as compiled-binary target; Node is canonical.
- **~25 providers, 9 wire APIs** via pi-ai.

## In the cross-agent landscape

- **pi is reused by [[openclaw|OpenClaw]] as its agent runtime substrate** — one of the few agents that's been adopted as a foundation by another agent. This validates the "small, predictable, stable substrate" thesis.
- The editorial restraint is a **deliberate design philosophy**, not unfinished software. The "refused features" list is in marketing-bullet form: they're choices.

## References

- [[pi-mono-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]
