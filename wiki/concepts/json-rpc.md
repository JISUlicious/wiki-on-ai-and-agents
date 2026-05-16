---
title: JSON-RPC
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - model-context-protocol-anthropic-2024.md
  - agent-client-protocol-zed-2025.md
status: draft
importance: medium
tags: []
---

JSON-RPC is a stateless, lightweight remote procedure call protocol that uses JSON for message encoding. Each request is a JSON object carrying a `method` name, a `params` payload, and an `id` used to correlate the eventual response; responses likewise carry either a `result` or an `error` object alongside the matching `id`. The protocol is transport-agnostic — it runs equally well over stdio, WebSockets, or HTTP — which makes it a natural substrate for agent-facing protocols where the wire format must be uniform across many language runtimes and process boundaries. Both [[mcp|MCP]] (the agent ↔ tool/data interface) and [[acp|ACP]] (the agent ↔ editor interface) define their entire message surface as JSON-RPC endpoints, so a single JSON-RPC parser can drive an agent's external integrations on both sides.

## Where it appears

- [[mcp]] — Model Context Protocol frames `tools/list`, `tools/call`, `resources/read`, etc. as JSON-RPC methods over stdio or SSE.
- [[acp]] — Agent Client Protocol uses JSON-RPC for the editor↔agent message bus (file ops, prompt streaming, permissions).

## References

- `model-context-protocol-anthropic-2024.md`
- `agent-client-protocol-zed-2025.md`
