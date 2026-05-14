---
title: "OpenClaw — Architecture & Implementation Notes"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - openclaw.md
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
  - "[[openclaw]]"
---

# OpenClaw — Architecture & Implementation Notes

**Source**: `sources/openclaw.md` (50KB technical document)
**Origin**: Architectural writeup of the OpenClaw multi-channel personal AI assistant (May 2026).

## Summary

[[openclaw|OpenClaw]] is a single-operator personal AI assistant that runs as a long-lived local **Gateway daemon** plus a fleet of optional client surfaces:

- CLI (`openclaw …`)
- Lit-based web Control UI
- macOS menu-bar app
- iOS / Android "node" apps
- In-process plugins for messaging channels: WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Matrix, Teams, Zalo, etc.

Built as a pnpm TypeScript monorepo (~thousand TS files in `src/`, 39 extensions, 52 skills). Re-uses Mario Zechner's `pi-mono` for the agent runtime and tools, but owns its own session model, gateway protocol, sandbox stack, ACP bridge, plugin SDK, and tool wiring.

Editorial stance from `VISION.md`:

> "OpenClaw is the AI that actually does things. It runs on your devices, in your channels, with your rules."

Single-operator, terminal-first, lean core, everything else as plugins/skills.

## Architecture

- **One daemon per host**: the Gateway, exposing a WebSocket control plane at `ws://127.0.0.1:18789`.
- **All clients (CLI, web UI, macOS app, mobile nodes) connect to the Gateway** for shared state.
- **Embedded pi-mono agent** for the runtime and core tools (7 tools).
- **OpenClaw's own ~13 tools** on top of pi-mono: browser, canvas, nodes, sessions_spawn, etc.
- **Tiered Docker sandbox**: `sandbox` / `sandbox-common` / `sandbox-browser` + Podman/Quadlet.
- **Memory**: inherited from pi (memory-core, memory-lancedb); single-slot.
- **Permissions**: per-session approval + `/elevated` break-glass + sandbox-level deny.
- **MCP**: refused in core; use `mcporter` bridge instead. Editorial decision.
- **ACP**: server-side yes; client-side via `acpx` — can embed Codex, Claude Code, Gemini, Pi.

## Key architectural facts

- **The "personal AI" form factor**: one user, many channels, persistent state.
- **Channel-aware**: each messaging channel (WhatsApp, Telegram, etc.) is a plugin that translates channel-specific events into the Gateway's protocol.
- **Embeds another agent (pi-mono)**: rare among the 5 documented agents; most own their full runtime.
- **Skills inherited from pi** (SKILL.md format) plus a ClawHub registry.
- **Sub-agents**: `sessions_spawn` exists, but VISION explicitly refuses agent-hierarchy frameworks. The single-operator stance pushes against multi-agent complexity.

## Entities Mentioned

- [[openclaw]] — the agent itself
- [[pi-mono]] — the embedded runtime
- [[claude-code]] — referenced in client embedding (acpx)

## Concepts Discussed

- [[llm-agent]] — multi-channel single-operator instance
- Gateway architecture
- Cross-agent embedding via ACP
- [[tool-use]]
- Capability containment via tiered sandbox

## References

_Original source: `sources/openclaw.md`_
_Also see [[cross-agent-comparison-2026|the cross-agent comparison]]._
