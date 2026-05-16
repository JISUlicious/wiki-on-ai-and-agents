---
title: Model Context Protocol (MCP)
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - model-context-protocol-anthropic-2024.md
status: complete
importance: high
tags:
  - 2024
---

# Model Context Protocol (MCP)

The **Model Context Protocol (MCP)** is an open standard for connecting LLM-based AI assistants to external data sources and tools. Introduced by [[anthropic|Anthropic]] on **November 25, 2024**, it standardizes the *agent ↔ tool/data* integration surface so that every AI app doesn't need a custom integration with every data source.

In the [[agent-three-layer-model]], MCP is the dominant **L3 implementation of the L2 action surface** — a cross-cutter that also touches L2 harness (via the auth layer per server) and L2 context assembly (servers can inject resources into the prompt).

## What it solves

The N×M integration problem. With N data sources and M AI applications, naive support requires N×M custom connectors. MCP collapses that to N+M:

- Each data source implements an MCP **server** once.
- Each AI application implements an MCP **client** once.
- All combinations work without further integration code.

Mirrors the role HTTP played for web pages, or JSON for data interchange — an interoperability protocol that becomes invisible infrastructure.

## Architecture

```
                ┌──────────────┐
                │  AI app /    │
                │  agent       │  ← MCP CLIENT
                │  (Claude,    │
                │  ChatGPT,    │
                │  Cursor, …)  │
                └──────┬───────┘
                       │ JSON-RPC
        ┌──────────────┼──────────────┐
        │              │              │
   ┌────▼────┐   ┌────▼────┐   ┌────▼────┐
   │ GDrive  │   │ GitHub  │   │ Custom  │  ← MCP SERVERS
   │ server  │   │ server  │   │ server  │
   └─────────┘   └─────────┘   └─────────┘
```

- **Servers** expose tools, resources, and prompts. Implemented once per data source.
- **Clients** discover and invoke server capabilities. Implemented once per AI app.
- **Transport**: JSON-RPC (stdio, SSE, HTTP).

## Deployment modes

- **Local**: server runs on the user's machine, no data leaves. Standard for Claude Desktop, Cursor, etc.
- **Remote**: centrally managed enterprise servers. Standardized auth + multi-tenant.

## Adoption timeline

| Date | Milestone |
|---|---|
| 2024-11-25 | Anthropic announces MCP; SDKs in Python + TypeScript |
| Feb 2025 | 1,000+ community MCP servers |
| Mar 2025 | OpenAI adopts MCP across products including ChatGPT desktop |
| Throughout 2025 | Google and Microsoft join; common cross-vendor infrastructure |
| Nov 2025 | Major spec update: async ops, statelessness, server identity, registry |
| Dec 2025 | Donated to **Agentic AI Foundation (AAIF)** under the Linux Foundation |

## Initial launch partners

- **Block** — early enterprise adopter; quoted by Anthropic in the announcement
- **Apollo** — enterprise adopter
- **Developer tools**: [[zed|Zed]], Replit, Codeium, Sourcegraph

## Relation to ACP

[[acp|ACP]] (Agent Client Protocol, from [[zed|Zed]]) and MCP are **complementary**, not competing:

| Protocol | Standardizes | Side |
|---|---|---|
| **MCP** | Agent ↔ tool/data integration | Model side |
| **ACP** | Agent ↔ editor integration | UI side |

A modern coding agent typically implements both: ACP to be hosted by any editor, MCP to connect to any tool or data source.

## Position in modern agent ecosystem

By 2026 MCP is universal among major AI assistants:
- [[claude-code]], [[opencode]], [[hermes-agent]] — first-class MCP support
- [[pi-mono]] — refuses MCP in core (extension required) — explicit minimalist stance
- [[openclaw]] — inherits the pi stance

This split makes MCP one of the **non-universal** features in the cross-agent comparison — see [[cross-agent-comparison-2026]].

## References

- [[model-context-protocol-anthropic-2024]] — the announcement source
- [[acp]] — complementary editor-side protocol
- [[agent-three-layer-model]] — MCP's place in the L2 action surface
- [[cross-agent-comparison-2026]] — where the 2026 agents differ on MCP adoption
- Canonical spec: https://modelcontextprotocol.io
