---
title: "Agent Client Protocol (ACP) — Zed"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - agent-client-protocol-zed-2025.md
authors:
  - "[[zed]]"
year: 2025
introduces:
  - "[[acp]]"
tags:
  - 2025
status: complete
importance: high
---

# Agent Client Protocol (ACP) — Zed

**Source**: `sources/agent-client-protocol-zed-2025.md` (fetched from https://zed.dev/acp)
**Publisher**: [[zed|Zed Industries]]
**Initial release**: August 2025

## Summary

The Agent Client Protocol ([[acp|ACP]]) is an open standard created by [[zed|Zed Industries]] that defines how AI coding agents communicate with editing environments. It solves the **agent-editor fragmentation problem**: before ACP, each agent (Claude Agent, Gemini CLI, GitHub Copilot, Codex) had to build N integrations for N editors (VS Code, JetBrains, Neovim, Emacs, etc.). ACP standardizes the interface — agent built once, runs in any ACP-compatible editor; editor built once, hosts any ACP-compatible agent.

**Technical foundation**: a minimal set of [[json-rpc|JSON-RPC]] endpoints that relay user requests to agents and render their responses. The editor is the **client** (exposing file-system access, codebase navigation, code review); the agent is the **server** (consuming those capabilities and emitting edits/observations). **Apache 2.0 licensed.**

**Privacy-first architecture**: "When using third-party agents, nothing touches our servers. We never store or train on your code without explicit consent." Code stays local.

**ACP vs [[mcp|MCP]]**: complementary, not competing. MCP standardizes **agent ↔ tool/data integration** (model-side). ACP standardizes **agent ↔ editor integration** (UI-side). An agent that speaks both ACP and MCP gets: any editor it can be hosted by × any tool it can connect to.

**Ecosystem (as of late 2025)**:
- **Editors implementing ACP**: [[zed|Zed]] (native), JetBrains (coming), Neovim (CodeCompanion, avante.nvim), Emacs (agent-shell), marimo, Obsidian, AionUi, Sidequery
- **Agents implementing ACP**: [[claude|Claude Agent]], Gemini CLI (Google), Cline, GitHub Copilot, Codex CLI (OpenAI), Goose (Square), [[opencode|opencode]], [[pi-mono|pi]], OpenHands, Augment Code, Blackbox AI, Mistral Vibe, Qwen Code, Kimi CLI, JetBrains Junie (coming)
- **Metrics**: Zed publicly tracks weekly sessions for the three most-used agents in Zed (Zed Agent, Claude Agent, Codex)

The first integration was [[claude|Claude]] via Anthropic's SDK adapter and Google's Gemini CLI (August 2025). JetBrains announced full ACP support in October 2025, signaling enterprise IDE buy-in.

## Key Points

- **Created by**: [[zed|Zed Industries]] in **August 2025**.
- **Solves the N×M agent-editor problem** — N agents × M editors → N+M integrations with shared protocol. Same logic as [[mcp|MCP]] but for the UI side.
- **JSON-RPC over a minimal endpoint set** — keeps the spec lean.
- **Apache 2.0** — fully open, with patent grant. Built by Zed, governed by community.
- **Privacy-first**: code stays local; "nothing touches our servers" when using third-party agents.
- **Complementary to MCP**: agents use ACP for editor integration AND MCP for tool/data integration.
- **Ecosystem (late 2025)**: ~15 editors + ~20 agents speaking ACP. Major IDE vendors (JetBrains) committed.
- **Both [[opencode|opencode]] and [[pi-mono|pi]] (covered elsewhere in this wiki) ship ACP support.**

## Entities Mentioned

- [[zed]] — creator and maintainer
- [[anthropic]] — Claude Agent is the most-used third-party agent in Zed
- [[openai]] — Codex CLI integrates via ACP
- [[opencode]] — implements ACP (matches existing wiki coverage)
- [[pi-mono]] — implements ACP

## Concepts Discussed

- [[acp]] — the protocol itself
- [[mcp]] — sister/complementary protocol
- [[llm-agent]] — what ACP hosts
- [[agent-three-layer-model]] — ACP sits in L2 action surface alongside MCP

## Notable Quotes

> "An open standard that enables any agent to integrate seamlessly with any editing environment."

> "When using third-party agents, nothing touches our servers. We never store or train on your code without explicit consent."

> "Created by Zed, grown by a community of editors and agents."

## References

_Original source: `sources/agent-client-protocol-zed-2025.md`_
_Page: https://zed.dev/acp_
_Blog announcements: https://zed.dev/blog/bring-your-own-agent-to-zed (Aug 2025), https://zed.dev/blog/acp-registry (registry launch), https://zed.dev/blog/acp-progress-report_
