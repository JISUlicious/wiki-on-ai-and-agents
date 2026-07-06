---
title: "MCP Server Architecture Patterns"
type: query
created: 2026-07-06
updated: 2026-07-06
sources:
  - mcp-server-architecture-patterns-rodrigues-2026.md
status: complete
importance: medium
tags:
  - 2026
---

# MCP Server Architecture Patterns

A synthesis of **[[mcp-server-architecture-patterns-rodrigues-2026|"MCP Server Architecture Patterns for LLM-Integrated Applications"]]** (Carson Rodrigues & Oysturn Vas, ICSME 2026 Industry Track) — a design-pattern catalog for **[[mcp|Model Context Protocol]]** servers.

## Why a pattern catalog

Hundreds of community MCP servers appeared within months of the protocol's Nov-2024 launch, but there was **no maintenance/architecture literature** on how they're actually structured in production. The authors distill patterns from a corpus of **15 independently developed servers** (5 production servers from the ANSYR voice-AI platform + 10 from the official `modelcontextprotocol/servers` registry) via two-cycle qualitative coding. Each pattern is written in classic **Gamma et al. (GoF) form** — context / problem / solution / consequences / known uses — and every MCP pattern has a **classical ancestor** (the novelty is the LLM-client constraint that reshapes it, not the skeleton).

## The five patterns

| Pattern | Ancestor | Problem it solves | Solution (essence) |
|---|---|---|---|
| **1. Resource Gateway** *(Data Facade / Context Provider)* | Facade / API gateway | LLM must read structured backend data, safely and stably across schema changes | Mediate **all** data access through one gateway: reads as **Resources**, unsafe parameterized queries as **Tools**, plus a **sanitization layer** stripping injected content before it reaches the LLM |
| **2. Tool Orchestrator** *(Action Hub / Workflow Facade)* | Facade / mediator | multi-system workflows (create ticket → notify → post) without the LLM knowing each API or handling partial failure | Expose **composite tools** that run all sub-calls internally and return one summary; the LLM sees a single operation |
| **3. Stateful Session Server** | Memento / session state | capabilities that need **per-session state** (memory KV store, git repo state, dialogue) | Keep server-side session state keyed per connection, rather than forcing state into the (stateless) LLM context |
| **4. Proxy Aggregator** | Proxy / API gateway | one operator must **aggregate many upstream servers/tools** but the client context can't hold them all | Partition/route tools to fit the context budget; front many servers behind one endpoint (directly motivated by the tool-count limit below) |
| **5. Domain-Specific Adapter** | Adapter (GoF) | a domain API / business logic doesn't map cleanly onto MCP primitives | Adapt the domain interface into MCP tools/resources with domain validation baked in |

The paper also documents **four anti-patterns** and cross-cutting concerns: **authentication, versioning, observability**.

## The quantitative backbone (why this is an *evidence-based* catalog)

- **Taxonomy reliability**: two independent LLM raters on **54 held-out servers** → Cohen's **κ = 0.76**, with three pattern-boundary ambiguities localized.
- **Transport overhead** (loopback p50): **stdio 0.01 ms** vs **streamable-http 0.39 ms**; cross-host modeled at **≈30 ms** baseline + protocol overhead → local stdio is ~40× cheaper, a real deployment lever.
- **Tool-count limit** (the empirical root of the Proxy Aggregator): tool-selection accuracy drops **below 90% at 10–15 tools** per context for **Claude Haiku 4.5**, and at **20–30 tools** for **Sonnet 4**. More tools ≠ more capable — context partitioning is a *correctness* concern, not just cost.

## Why it matters (positioning)

- It's the [[mcp|MCP]] ecosystem's first **software-engineering** treatment — moving MCP from "wire protocol spec" to **architectural practice** (the way design patterns did for OOP). Complements the [[agent-skills]] / [[agent-three-layer-model|L2 action-surface]] view: MCP standardizes the *protocol*; these patterns standardize the *server shapes* behind it.
- The tool-count finding is a concrete data point for the recurring "how many tools is too many?" question, and a systems-level echo of context-management concerns in [[fundamental-components-of-harness|harness design]].
- The Proxy Aggregator + tool-count limit connect to **[[learned-orchestration]]** and dynamic tool routing — when one server can't expose everything, something upstream must select.

## References

- [[mcp-server-architecture-patterns-rodrigues-2026]] — the source (arXiv:2606.30317, ICSME 2026 Industry Track).
- [[mcp]] — the protocol these patterns build on.
