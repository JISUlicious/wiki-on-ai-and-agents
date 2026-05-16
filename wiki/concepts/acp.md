---
title: Agent Client Protocol (ACP)
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - agent-client-protocol-zed-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Agent Client Protocol (ACP)

The **Agent Client Protocol (ACP)** is an open standard from [[zed|Zed Industries]] (released **August 2025**) that defines how AI coding agents communicate with editing environments. ACP is to **agent-editor integration** what [[mcp|MCP]] is to **agent-tool integration**: a thin JSON-RPC layer that decouples N agents × M editors into N+M implementations.

In the [[agent-three-layer-model]], ACP sits at **L2 action surface** alongside [[mcp|MCP]] and typed tool APIs — it's the protocol layer that lets agents act in editor environments.

## What it solves

Before ACP: every AI coding agent (Claude Agent, Gemini CLI, Codex, Copilot) had to be ported individually to every editor (VS Code, JetBrains, Neovim, Emacs). Pairwise integrations bloat — both directions of the market suffer.

ACP collapses this to one specification: any agent that speaks ACP plugs into any ACP-compatible editor. The agent-side ecosystem and the editor-side ecosystem evolve independently.

## Architecture

```
       ┌──────────────────┐
       │      EDITOR      │
       │  (Zed, JetBrains │  ← ACP CLIENT
       │   Neovim, Emacs, │     exposes file I/O,
       │   marimo, …)     │     codebase nav, code review
       └────────┬─────────┘
                │ JSON-RPC
       ┌────────▼─────────┐
       │      AGENT       │  ← ACP SERVER
       │  (Claude Agent,  │     consumes editor capabilities,
       │   Gemini CLI,    │     emits edits + observations
       │   Codex, opencode,│
       │   pi, …)         │
       └──────────────────┘
```

- **Editor = client**: hosts the agent, provides file/codebase/review capabilities.
- **Agent = server**: emits edits, observations, tool calls.
- **Transport**: minimal JSON-RPC endpoints (kept lean by design).
- **License**: Apache 2.0 with patent grant.

## Privacy-first

> "When using third-party agents, nothing touches our servers. We never store or train on your code without explicit consent."

Code stays local. Distinguishes ACP-hosted workflows from cloud-only agent UIs.

## ACP vs MCP

Complementary protocols, not substitutes:

| Protocol | Side | What's standardized |
|---|---|---|
| [[mcp]] | Model side | Agent ↔ tool/data sources |
| ACP | UI side | Agent ↔ editor environment |

A modern coding agent typically speaks **both** — MCP for tools/data, ACP for the editor surface.

## Ecosystem (as of late 2025)

**Editors implementing ACP:**
- [[zed|Zed]] — native (originating platform); supports agent-following + real-time editing
- JetBrains IDEs (IntelliJ, PyCharm, etc.) — full integration announced October 2025
- Neovim — via CodeCompanion / avante.nvim plugins
- Emacs — via agent-shell
- marimo — Python notebook environment
- Obsidian — side-panel agent chat
- AionUi, Sidequery (browser-based, coming)

**Agents implementing ACP:**
- [[claude|Claude Agent]] (via Anthropic SDK adapter)
- Gemini CLI (Google) — *first integration, August 2025*
- Codex CLI (OpenAI)
- GitHub Copilot
- [[opencode]], [[pi-mono|pi]]
- Cline, OpenHands, Goose (Square)
- Augment Code, Blackbox AI, Docker cagent
- Mistral Vibe, Qwen Code, Kimi CLI
- JetBrains Junie (coming)
- Specialized: aizen, Tidewave, Stakpak, DeepChat

**Metrics**: Zed publicly tracks weekly sessions for the three most-used agents in Zed (Zed Agent, Claude Agent, Codex CLI).

## Recent developments

- **Aug 2025**: Initial release; Gemini CLI as the first integration partner
- **Oct 2025**: JetBrains collaboration — enterprise IDE buy-in
- Late 2025: Registry launch (https://zed.dev/blog/acp-registry) — register your agent or editor once, available to all ACP clients/agents

## Position in cross-agent landscape

Among the 2026 agents covered in this wiki ([[cross-agent-comparison-2026]]):
- [[opencode]] — first-class ACP support; ships an ACP client component
- [[pi-mono]] — implements ACP via extension
- [[hermes-agent]] — has ACP support
- [[claude-code]] — not ACP-in-tree (Anthropic's own runtime), but Claude Agent (the model) is available *through* ACP via the SDK adapter
- [[openclaw]] — ships ACP server + client

ACP is therefore **partially universal** — implemented by 4–5 of the 5 agents in some form. Closer to baseline than MCP (which has a 3:2 split).

## References

- [[agent-client-protocol-zed-2025]] — the source page
- [[mcp]] — sister protocol on the model side
- [[zed]] — creator
- [[agent-three-layer-model]] — ACP's place in the L2 action surface
- [[cross-agent-comparison-2026]] — adoption across the 5 documented agents
- Canonical page: https://zed.dev/acp
