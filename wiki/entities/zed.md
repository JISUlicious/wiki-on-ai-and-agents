---
title: Zed Industries
type: entity
created: 2026-05-16
updated: 2026-05-16
sources:
  - agent-client-protocol-zed-2025.md
status: draft
importance: medium
tags:
  - org
---

# Zed Industries

**Zed Industries** is the company behind the Zed code editor, a high-performance Rust-written collaborative editor that emphasizes speed and modern developer workflow. In the LLM-agent era, Zed has become a central player by creating and shepherding the [[acp|Agent Client Protocol]] — an open standard for integrating AI coding agents with editing environments.

Zed's design philosophy bridges the gap between the local-editor tradition (Vim, Emacs, JetBrains) and the cloud-collaborative model (VS Code Live Share, Figma) — a Rust binary fast enough to feel native, but multiplayer-by-default and AI-host-ready.

## Role in the agent ecosystem

- **Created [[acp|ACP]] (August 2025)**: Apache 2.0 open standard for agent-editor interop. Cited as the wiki's L3 implementation of the L2 action surface alongside [[mcp|MCP]].
- **Hosts multiple agents** including Anthropic's Claude Agent (via SDK adapter), Google's Gemini CLI, OpenAI's Codex CLI, [[opencode]], [[pi-mono|pi]], and others.
- **MCP launch partner (2024)**: Zed was one of the first developer tools to implement Anthropic's [[mcp|MCP]] alongside Replit, Codeium, and Sourcegraph.
- **ACP Registry (late 2025)**: published a registry so agents and editors can register once and be available to all ACP-compatible counterparts.

## Notable work

- [[acp]] — open protocol for agent-editor integration
- Zed editor — Rust-written collaborative code editor with native AI agent hosting
- [[agent-client-protocol-zed-2025]] — the ACP source documenting their approach

## References

- [[agent-client-protocol-zed-2025]]
- [[acp]]
- [[mcp]] — sister protocol Zed implemented as early launch partner
- Website: https://zed.dev
