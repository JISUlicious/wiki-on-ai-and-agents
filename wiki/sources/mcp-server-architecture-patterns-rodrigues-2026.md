---
title: "MCP Server Architecture Patterns for LLM-Integrated Applications"
type: source
created: 2026-07-06
updated: 2026-07-06
sources:
  - mcp-server-architecture-patterns-rodrigues-2026.md
arxiv_id: "2606.30317"
authors:
  - Carson Rodrigues
  - Oysturn Vas
first_author: Carson Rodrigues
year: 2026
venue: ICSME 2026 (Industry Track)
tags: [2026]
status: complete
importance: medium
---

# MCP Server Architecture Patterns for LLM-Integrated Applications

*Credit: NLP newsletter (2026-06-28–07-05).*

## Summary

An industry-experience paper that catalogs recurring architectural patterns for **Model Context Protocol (MCP)** servers — the client–server standard (introduced by Anthropic, Nov 2024) that exposes tools, resources, and prompts to any MCP-compatible LLM client. With hundreds of community MCP servers appearing within months but no maintenance literature on how the ecosystem is structured in production, the authors distill **five recurring server patterns** from a corpus of fifteen independently developed servers (five production servers from the ANSYR voice-AI platform plus ten public servers from the official MCP registry).

Each pattern is written in the classic Gamma et al. design-pattern form (context, problem, solution, consequences). The paper also documents four anti-patterns and cross-cutting concerns (authentication, versioning, observability), then backs the taxonomy with quantitative measurements.

## Key points

- **Five MCP server patterns**: Resource Gateway, Tool Orchestrator, Stateful Session Server, Proxy Aggregator, and Domain-Specific Adapter — each described in Gamma-style context/problem/solution/consequences form.
- Corpus of **fifteen** servers (5 production from ANSYR voice AI + 10 from the official MCP registry); also documents **four anti-patterns** plus cross-cutting concerns (auth, versioning, observability).
- Taxonomy inter-rater reliability across two independent LLM raters on **54 held-out servers**: Cohen's **κ = 0.76**, localizing three pattern-boundary ambiguities.
- Transport overhead measured on loopback: **stdio 0.01 ms p50** vs **streamable-http 0.39 ms p50**; cross-host modeled at ≈30 ms p50 baseline plus protocol overhead.
- Tool-count study: accuracy drops below **90%** between 10–15 tools per context for Claude Haiku 4.5, and between 20–30 tools for Sonnet 4.
- Positioned as an industry experience paper (ICSME 2026 Industry Track); code, corpus, and prompts released on GitHub.

## Concepts & entities

- [[mcp]] — the protocol these server patterns are built on; this source is a practitioner-oriented pattern catalog for MCP server design.

## References

- [arXiv:2606.30317](https://arxiv.org/abs/2606.30317) — "MCP Server Architecture Patterns for LLM-Integrated Applications", Carson Rodrigues, Oysturn Vas, 2026 (ICSME 2026 Industry Track).
