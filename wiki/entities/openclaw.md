---
title: OpenClaw
type: entity
created: 2026-05-14
updated: 2026-11-02
sources:
  - openclaw.md
  - comparison.md
family: agent-product
parameters: "n/a (inherits pi-mono provider stack)"
release_date: 2026-01-30
status: complete
importance: medium
tags:
  - product
---

# OpenClaw

> [!warning] Contradiction
> This page, written from May 2026 docs, says OpenClaw embeds pi-mono as its runtime. Commit `bb46b79d3c1` (2026-05-27) internalized that runtime. `THIRD_PARTY_NOTICES.md` now says portions were adapted from Pi, and only `pi-tui` remains a dependency. Earlier names were Warelay, Clawdis, Clawdbot and Moltbot. See [[agent-products-repo-history-2026]].

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
- **Embeds [[pi-mono|pi-mono]] as the agent runtime** — uses pi's tools + agent loop, but owns its own session model, gateway protocol, sandbox stack, ACP bridge, plugin SDK. **Historical:** true from 2025-12-05 to 2026-05-27. See the note below.
- **Tools**: pi's 7 + OpenClaw's ~13 (browser, canvas, nodes, sessions_*, ...).
- **Tiered Docker sandbox**: `sandbox` / `sandbox-common` / `sandbox-browser` + Podman/Quadlet.
- **Permissions**: per-session approval + `/elevated` break-glass + sandbox-level deny.
- **MCP**: refused in core; uses `mcporter` bridge if you want it.
- **ACP**: server-side yes; client-side via `acpx` — can *embed* Codex, [[claude-code|Claude Code]], Gemini, pi from inside OpenClaw.

## Distinguishing features

- **Single-operator, multi-channel**: the "personal AI assistant" form factor done well. Most other documented 2026 agents are coding-focused.
- **Embeds pi-mono**: rare among documented agents to take another agent's runtime wholesale. Validates pi's "small, stable substrate" pitch.
- **Channel pluginization**: each messaging channel (WhatsApp, Telegram, etc.) is an in-process plugin that translates platform events into Gateway protocol messages.
- **Refuses hierarchical multi-agent frameworks** while still allowing `sessions_spawn` — the VISION pushes against agent-of-agents complexity.

**Goal and trigger features:** `/goal` (2026-05-29) lets the working model or operator mark completion; no separate judge was found in goal-named files. Standing orders, standing intents and a Proactive check job cover proactive work. See [[agent-goal-loops-extension-docs-2026]].

## References

- [[openclaw-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]

## Related 2026 sources

- [[illicit-distillation]] — named twice in Anthropic's 2026 report: Xiaomi allegedly replayed its users' OpenClaw sessions through Claude for training data, and attackers prompt-injected OpenClaw deployments to steal API keys
- [[skill2env-capability-oriented-environment-synthesis-allspark-2026]] — Skill2Env: 3K public skills hardened into executable tasks; Terminal-Bench 2.1 44.9→58.4 (no distillation control)
