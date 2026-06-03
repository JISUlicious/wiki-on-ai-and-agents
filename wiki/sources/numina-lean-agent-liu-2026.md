---
title: "Numina-Lean-Agent: An Open and General Agentic Reasoning System for Formal Mathematics"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - numina-lean-agent-liu-2026.md
arxiv_id: "2601.14027"
authors:
  - Junqi Liu
  - Zihao Zhou
  - Zekai Zhu
  - "(et al., Project Numina)"
first_author: Junqi Liu
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Numina-Lean-Agent: An Open and General Agentic Reasoning System for Formal Mathematics

[arXiv:2601.14027](https://arxiv.org/abs/2601.14027) · Liu, Zhou, Zhu et al. (UCAS / Project Numina / Cambridge / Edinburgh) · 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary
Agentic systems have become the dominant paradigm for formal theorem proving, but existing approaches lean on task-specific pipelines and specially-trained formal provers, hurting flexibility and reproducibility. This paper proposes using a *general coding agent* directly as a formal math reasoner. Motivation: a coding agent is a natural interface for diverse reasoning tasks beyond proving, performance improves simply by swapping the base model (no training), and MCP enables plug-and-play addition and autonomous invocation of specialized tools.

The concrete system, Numina-Lean-Agent, pairs Claude Code with a custom Numina-Lean-MCP server (which wraps Lean-LSP-MCP for live Lean interaction, theorem retrieval, informal proving, and auxiliary tools). With Claude Opus 4.5 as the base model it solves all 12/12 Putnam 2025 problems — matching the best closed-source system (AxiomProver) — and the authors further formalize the Brascamp–Lieb theorem in collaboration with mathematicians, demonstrating generality. Code and solutions are released openly.

## Key points
- Paradigm shift: use a general coding agent (Claude Code) as the formal reasoner instead of a bespoke trained prover.
- No-training improvement path: capability scales by replacing the underlying base model rather than retraining a prover.
- Numina-Lean-MCP exposes Lean interaction (via Lean-LSP-MCP), theorem retrieval, informal proving, and auxiliary reasoning tools; the agent autonomously selects and calls them.
- Result: 12/12 on Putnam 2025 with Claude Opus 4.5, matching closed-source AxiomProver (SOTA).
- Generality demo: human-in-the-loop formalization of the Brascamp–Lieb theorem, beyond benchmark proving.
- Open release of the agent and all solutions; contrasts with closed pipelines (HILBERT, Seed-Prover 1.5, AxiomProver).

## Concepts & entities
- [[claude-code]]
- [[mcp]]
- [[llm-agent]]
- [[code-as-harness]]
- [[tool-use]]
- [[anthropic]]

## References
- arXiv:2601.14027
