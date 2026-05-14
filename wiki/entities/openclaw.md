---
title: OpenClaw
type: entity
created: 2026-05-14
updated: 2026-05-14
sources:
  - openclaw.md
  - comparison.md
family: agent-product
parameters: "n/a (inherits pi-mono provider stack)"
release_date: 2025
status: complete
importance: medium
tags:
  - product
---

# OpenClaw

OpenClaw is a single-operator, multi-channel personal AI assistant. Runs as a long-lived local **Gateway** daemon plus a fleet of optional client surfaces, all connecting back to the Gateway over WebSocket. Editorial stance from `VISION.md`:

> "OpenClaw is the AI that actually does things. It runs on your devices, in your channels, with your rules."

## Surfaces

- CLI (`openclaw …`)
- Lit-based web Control UI
- macOS menu-bar app
- iOS / Android "node" apps
- Messaging-channel plugins: WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Matrix, Teams, Zalo, ...

## Architecture

- **pnpm TypeScript monorepo**, ~thousand TS files in `src/`, 39 extensions, 52 skills.
- **One Gateway daemon per host**, exposing `ws://127.0.0.1:18789` as the control plane. All clients connect to it.
- **Embeds [[pi-mono|pi-mono]] as the agent runtime** — uses pi's tools + agent loop, but owns its own session model, gateway protocol, sandbox stack, ACP bridge, plugin SDK.
- **Tools**: pi's 7 + OpenClaw's ~13 (browser, canvas, nodes, sessions_*, ...).
- **Tiered Docker sandbox**: `sandbox` / `sandbox-common` / `sandbox-browser` + Podman/Quadlet.
- **Permissions**: per-session approval + `/elevated` break-glass + sandbox-level deny.
- **MCP**: refused in core; uses `mcporter` bridge if you want it.
- **ACP**: server-side yes; client-side via `acpx` — can *embed* Codex, Claude Code, Gemini, pi from inside OpenClaw.

## Distinguishing features

- **Single-operator, multi-channel**: the "personal AI assistant" form factor done well. Most other documented 2026 agents are coding-focused.
- **Embeds pi-mono**: rare among documented agents to take another agent's runtime wholesale. Validates pi's "small, stable substrate" pitch.
- **Channel pluginization**: each messaging channel (WhatsApp, Telegram, etc.) is an in-process plugin that translates platform events into Gateway protocol messages.
- **Refuses hierarchical multi-agent frameworks** while still allowing `sessions_spawn` — the VISION pushes against agent-of-agents complexity.

## References

- [[openclaw-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]
