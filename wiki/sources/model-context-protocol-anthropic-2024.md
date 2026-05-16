---
title: "Introducing the Model Context Protocol"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - model-context-protocol-anthropic-2024.md
authors:
  - "[[anthropic]]"
year: 2024
introduces:
  - "[[mcp]]"
tags:
  - 2024
status: complete
importance: high
---

# Introducing the Model Context Protocol

**Source**: `sources/model-context-protocol-anthropic-2024.md` (fetched from https://www.anthropic.com/news/model-context-protocol)
**Publisher**: [[anthropic|Anthropic]]
**Date**: 2024-11-25 (initial announcement); updated content reflects 2025 ecosystem evolution

## Summary

The Model Context Protocol ([[mcp|MCP]]) is an open-source standard that [[anthropic|Anthropic]] announced on November 25, 2024 to standardize how LLM-based AI assistants connect to external data sources and tools. Before MCP, every AI assistant ↔ data source combination required custom integration code — an N×M problem that scaled poorly. MCP reduces it to N+M: each tool implements an MCP server once, each AI client implements MCP support once, and all combinations work.

The protocol uses a **client-server architecture** over [[json-rpc|JSON-RPC]]. **MCP servers** expose data sources (databases, APIs, business apps); **MCP clients** are the AI applications consuming that data ([[claude]], Claude Desktop, etc.). MCP supports both **local deployment** (private servers on user's machine — no data leaves) and **remote deployment** (centrally managed enterprise servers).

Anthropic launched MCP with: the spec, SDKs (Python + TypeScript first; C#, Java soon after), Claude Desktop integration, and pre-built servers for Google Drive, Slack, GitHub, Git, Postgres, Puppeteer.

**Adoption trajectory through 2025–2026**: 1,000+ community servers by Feb 2025 → [[openai|OpenAI]] officially adopts MCP March 2025 → Google + Microsoft join → MCP evolves from vendor-led spec into common infrastructure. Major spec updates Nov 2025 (async operations, statelessness, server identity, registry). **December 2025: Anthropic donates MCP to the Agentic AI Foundation (AAIF) under the Linux Foundation** — completing the transition to community-owned infrastructure.

This source is **foundational for understanding the agent action surface** ([[agent-three-layer-model]] L2): MCP is the dominant L3 implementation of the typed action surface for connecting LLM agents to external tools and data.

## Key Points

- **Announced**: 2024-11-25 by [[anthropic|Anthropic]].
- **Solves the N×M integration problem** — N tools × M AI apps → N+M with a shared protocol.
- **Architecture**: client-server over JSON-RPC. Clients consume; servers expose data/tools.
- **Two deployment modes**: local (privacy-preserving) and remote (enterprise-managed).
- **Initial launch partners**: Block, Apollo, [[zed]], Replit, Codeium, Sourcegraph.
- **OpenAI adoption (Mar 2025)**: cemented MCP as cross-vendor standard.
- **Donated to Agentic AI Foundation (Linux Foundation, Dec 2025)**: open-governance infrastructure.
- **Quote (Dhanji R. Prasanna, Block CTO)**: *"Open technologies like the Model Context Protocol are the bridges that connect AI to real-world applications, ensuring innovation is accessible, transparent, and rooted in collaboration."*

## Entities Mentioned

- [[anthropic]] — originator
- [[openai]] — adopted Mar 2025
- [[zed]] — early ecosystem partner; also created the complementary [[acp|ACP]]

## Concepts Discussed

- [[mcp]] — the protocol itself
- [[llm-agent]] — primary consumers
- [[tool-use]] — what MCP standardizes
- [[acp]] — complementary protocol for agent-editor integration
- [[agent-three-layer-model]] — MCP sits in L2 action surface (with cross-cutter into harness and context-assembly)

## Notable Quotes

> "An open standard that enables developers to build secure, two-way connections between their data sources and AI-powered tools." — Anthropic announcement

> "Open technologies like the Model Context Protocol are the bridges that connect AI to real-world applications, ensuring innovation is accessible, transparent, and rooted in collaboration." — Dhanji R. Prasanna, CTO at Block

## References

_Original source: `sources/model-context-protocol-anthropic-2024.md`_
_Canonical spec: https://modelcontextprotocol.io_
_Announcement: https://www.anthropic.com/news/model-context-protocol_
