---
title: "Stop Comparing LLM Agents Without Disclosing the Harness"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026.md
arxiv_id: "2605.23950"
authors:
  - Yunbei Zhang
  - et al.
first_author: Yunbei Zhang
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Stop Comparing LLM Agents Without Disclosing the Harness

> Surfaced while researching harness-vs-model performance attribution.

## Summary

A position paper (Zhang et al., Tulane/Rutgers/Virginia Tech, 2026) arguing that for long-horizon agent tasks among models of comparable frontier capability, the **execution harness** — the infrastructure layer governing context construction, tool mediation, orchestration, and verification — is often a stronger determinant of [[agent-benchmark]] performance than the model it wraps. It formalizes the **Binding Constraint Thesis**: in this regime performance variance is governed more by harness configuration than by model choice, so leaderboards that report a single number per {model, benchmark} pair systematically misattribute harness-level gains to model improvements.

The paper defends the thesis three ways: (1) a control-theoretic formalization treating the harness as the controller of a closed-loop dynamical system and the [[llm-agent]] as the stochastic policy it governs; (2) empirical evidence from benchmarks, deployments, and a variance decomposition; and (3) a harness-aware evaluation framework with a disclosure standard. Until harness specs are disclosed, it argues, agent leaderboard comparisons should be treated as incomplete and potentially misleading.

## Key points

- **Scaffold-only variation is large.** Independent third-party monitoring reports up to **11 pts** scaffold-only variation for GPT-5 and **15 pts** for Kimi K2 Thinking on SWE-bench Verified; HAL reports single-model swings up to ~48 pts on SWE-bench Verified Mini.
- **Held-fixed-model examples.** Same model, harness-only change raises [[terminal-bench]] 2 pass@1 from **69.7% → 77.0%**; Claude Opus 4.5 goes **45.9% → 55.4%** on SWE-bench Pro moving from SEAL scaffold to Claude Code.
- **Ranking reversals.** Adding a single search subagent (WarpGrep) flips the SWE-bench Pro ordering between MiniMax 2.5 and Claude Opus 4.6, despite Opus ranking higher elsewhere.
- **Closed-loop framing.** Agents are discrete-time closed-loop systems; the harness is the controller (stability V(s_t), context drift δ_t, control lag τ), explaining why small harness changes can exceed model-substitution effects.
- **Attribution gap.** Harness swings routinely dwarf the 2–4 pt shifts papers report as meaningful model advances; the same composite score is misread as a model property, a benchmark property, and an agent property.
- **Proposed fix.** Either a **locked-harness** protocol (one specified harness for all models) or a **factorial** protocol (harness varied as a controlled factor with reported variance components), plus a harness disclosure standard.

## Concepts & entities

- [[agentic-harness-engineering]] — the discipline this paper argues evaluation must account for
- [[agent-benchmark]] · [[llm-agent]] · [[terminal-bench]]
- [[swe-bench]] (and SWE-bench Verified / Pro variants, plain text)
- Related: [[code-as-harness]] · [[code-generation]]

## References

- [arXiv:2605.23950](https://arxiv.org/abs/2605.23950) — "Stop Comparing LLM Agents Without Disclosing the Harness", Zhang et al., 2026
- Companion harness-attribution work: [[agentic-harness-engineering-lin-2026]], [[harness-updating-is-not-harness-benefit-lin-2026]], [[scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026]]
