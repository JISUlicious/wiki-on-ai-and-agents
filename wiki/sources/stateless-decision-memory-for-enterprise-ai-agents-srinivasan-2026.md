---
title: "Stateless Decision Memory for Enterprise AI Agents"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - stateless-decision-memory-for-enterprise-ai-agents-srinivasan-2026.md
arxiv_id: "2604.20158"
authors:
  - Vasundra Srinivasan
first_author: Vasundra Srinivasan
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Stateless Decision Memory for Enterprise AI Agents

*Surfaced via the alphaXiv / NLP-newsletter digest.*

[arXiv:2604.20158](https://arxiv.org/abs/2604.20158) — Vasundra Srinivasan, 2026.

## Summary

This paper asks why enterprise deployment of long-horizon decision agents in regulated domains (underwriting, claims adjudication, clinical review, tax examination) runs overwhelmingly on plain retrieval-augmented pipelines despite a decade of more sophisticated *stateful* memory architectures. The answer: regulated deployment is load-bearing on four systems properties — **deterministic replay, auditable rationale, multi-tenant isolation, and statelessness for horizontal scale** — that stateful memory violates by construction. The author proposes **Deterministic Projection Memory (DPM)**, which treats agent memory as an append-only event log plus a single task-conditioned projection at decision time. The core claim is not a new accuracy win but that *statelessness* explains enterprise's preference for weaker-but-replayable retrieval, and that DPM attains it without retrieval's decisioning penalty.

## Key points

- Identifies four under-evaluated systems requirements for regulated agents: deterministic replay, auditable rationale, multi-tenant isolation, and stateless horizontal scalability.
- Stateful memory architectures violate at least one of these "by construction," and the violation compounds as deployment matures.
- **DPM** = append-only event log + a single task-conditioned projection computed at decision time (one LLM call instead of N across the trajectory).
- Matches summarization-based memory at generous budgets; at a **20x compression ratio** it improves factual precision (+0.52, Cohen's h=1.17, p=0.0014) and reasoning coherence (+0.53, h=1.13, p=0.0034); n=10, paired permutation.
- **7–15x faster** than the stateful baseline by collapsing N trajectory calls into one decision-time projection.
- Contributes **TAMS**, a practitioner heuristic for memory-architecture selection, plus a failure analysis of stateful memory under enterprise conditions.

## Concepts & entities

- [[memory-management]]
- [[retrieval-augmented-generation]]
- [[llm-agent]]

## References

- [arXiv:2604.20158](https://arxiv.org/abs/2604.20158)
- Local source: `sources/stateless-decision-memory-for-enterprise-ai-agents-srinivasan-2026.md`
