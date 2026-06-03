---
title: "AlphaEval: Evaluating Agents in Production"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - alphaeval-evaluating-agents-in-production-lu-2026.md
arxiv_id: "2604.12162"
authors:
  - Pengrui Lu
  - Bingyu Xu
  - Wenjun Zhang
  - Shengjia Hua
  - Pengfei Liu
first_author: Pengrui Lu
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# AlphaEval: Evaluating Agents in Production

[arXiv:2604.12162](https://arxiv.org/abs/2604.12162) — Lu, Xu, Zhang, Hua, …, Liu (SII, MiraclePlus, SJTU, GAIR et al.), 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Agent deployment in commercial settings has outpaced evaluation methodology. Existing benchmarks use retrospectively curated tasks with well-specified requirements and deterministic metrics — conditions that diverge sharply from production, where requirements carry implicit constraints, inputs are heterogeneous multi-modal documents with fragmented information, tasks need undeclared domain expertise, outputs are long-horizon professional deliverables, and success is judged by domain experts whose standards evolve. **AlphaEval** is a production-grounded benchmark of **94 tasks** sourced from seven companies deploying AI agents in their core business, spanning six **O*NET** occupational domains.

Unlike model-centric benchmarks, AlphaEval evaluates complete agent *products* (Claude Code, Codex, GitHub Copilot, Cursor) as commercial systems across 14 model–scaffold configurations, using a multi-paradigm evaluation framework (LLM-as-judge, reference metrics, formal verification, rubric assessment, automated UI testing). The best config (Claude Code + Opus 4.6) reaches only **64.41/100**, exposing a large research–production gap. The paper also contributes a reusable requirement-to-benchmark construction framework.

## Key points

- **94 production tasks** from seven companies across six O*NET occupational domains, preserving real-world complexity (ambiguous specs, multi-modal inputs, long-horizon deliverables).
- **Product-level, not model-level evaluation:** scores full agent products (Claude Code, Codex, Copilot, Cursor) in 14 model–scaffold configurations.
- **Scaffold matters as much as model:** the same Opus 4.6 scores 64.41 via Claude Code but only 53.45 via Codex — an 11-point spread.
- **Best score only 64.41/100,** quantifying the research-to-production gap.
- **Multi-paradigm framework** composes LLM-as-judge, reference-driven metrics, formal verification, rubrics, and automated UI testing per domain.
- **Requirement-to-benchmark construction framework:** a reproducible, modular pipeline any organization can use to turn its own production requirements into executable benchmarks; six production-specific failure modes identified.

## Concepts & entities

- [[agent-benchmark]] — AlphaEval is a production-grounded agent benchmark.
- [[llm-agent]] — evaluates complete commercial LLM-agent products.
- [[gaia]] — contrast point: a prior research-style agent benchmark vs. production grounding.
- [[claude-code]] — one of the evaluated agent products (best-scoring scaffold).

## References

- [arXiv:2604.12162](https://arxiv.org/abs/2604.12162)
