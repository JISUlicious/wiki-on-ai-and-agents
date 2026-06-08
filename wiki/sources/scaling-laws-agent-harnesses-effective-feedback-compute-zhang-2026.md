---
title: "Scaling Laws for Agent Harnesses via Effective Feedback Compute"
type: source
created: 2026-06-08
updated: 2026-06-08
sources:
  - scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026.md
arxiv_id: "2605.29682"
authors:
  - Xuanliang Zhang
  - Dingzirui Wang
  - et al.
first_author: Xuanliang Zhang
year: 2026
venue: Harbin Institute of Technology (preprint)
tags: [2026]
status: complete
importance: medium
---

# Scaling Laws for Agent Harnesses via Effective Feedback Compute

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-05-31–06-07).

## Summary

Agent-harness performance is a form of test-time scaling, but existing analyses parameterize it by *raw* expenditure — tokens, tool calls, wall time, cost — which fails to distinguish useful feedback from redundant or unstable interaction. This paper (Harbin Institute of Technology) introduces **Effective Feedback Compute (EFC)**, a trace-level scaling coordinate that credits feedback only when it is *informative, valid, non-redundant, and retained* for later decisions, then normalizes it by **task demand** (`Dtask`) so tasks with different feedback requirements are comparable.

Across synthetic controllable tasks, executable code tasks, real benchmark traces, held-out splits, and a prospective validation batch, EFC coordinates predict failure rates far better than raw-compute baselines. In controlled scaling, raw tokens and tool calls explain little (R²≈0.33 and 0.42); a strong multivariate SAS baseline reaches 0.88, while Oracle-EFC normalized by task demand reaches **R²≈0.99**. Matched-budget interventions are decisive: at *fixed* raw cost and tool calls, improving feedback quality raises success from 0.27 to 0.90. On mixed real traces, NRS-EFC/Dtask reaches R²≈0.92 (raw compute near-zero/negative) and remains best on a prospective holdout (0.85). The takeaway: harness scaling is governed by how efficiently raw budget converts into durable, task-sufficient feedback — quality over quantity.

## Key points

- Introduces **Effective Feedback Compute (EFC)**: feedback counts only when informative, valid, non-redundant, and retained.
- Normalizing EFC by **task demand (Dtask)** makes scaling coordinates comparable across tasks.
- **Raw budget predicts weakly**: tokens R²≈0.33, tool calls R²≈0.42; Oracle-EFC/Dtask reaches R²≈0.99 (SAS baseline 0.88).
- **Matched-budget result**: at fixed cost and tool calls, better feedback quality lifts success 0.27 → 0.90 — quality beats quantity.
- Holds on real traces (NRS-EFC/Dtask R²≈0.92) and a prospective holdout (R²≈0.85), where raw compute fits near zero or negative.

## Concepts & entities

- [[scaling-laws]] — a feedback-quality scaling coordinate for agent harnesses (test-time scaling)
- [[code-as-harness]] — harness decides tool use, feedback, memory, verification, stopping
- [[agentic-harness-engineering]] — EFC as a design objective for engineering harnesses
- [[reasoning-effort]] — converting inference-time budget into effective feedback

## References

- [arXiv:2605.29682](https://arxiv.org/abs/2605.29682) — Zhang, Wang, et al., 2026
- Source text: `sources/scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026.md`
