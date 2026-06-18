---
title: "Self-Harness: Harnesses That Improve Themselves"
type: source
created: 2026-06-18
updated: 2026-06-18
sources:
  - self-harness-harnesses-that-improve-themselves-zhang-2026.md
arxiv_id: "2606.09498"
authors:
  - Hangfan Zhang
  - et al.
first_author: Hangfan Zhang
year: 2026
venue: arXiv preprint (Shanghai AI Laboratory)
tags:
  - 2026
status: complete
importance: medium
---

# Self-Harness: Harnesses That Improve Themselves

Surfaced via the alphaXiv weekly digest (2026-06-17).

## Summary

Self-Harness is a paradigm in which a *fixed* LLM-based agent autonomously improves its own operating harness — the surrounding system of system prompts, tools, runtime mechanisms, verification rules, orchestration logic, and failure-recovery procedures that mediate the model's interaction with the environment. The motivating observation is that effective harness design is inherently model-specific (different models have distinct error modes and tool-use habits), yet harnesses are still hand-engineered by human experts, which scales poorly as LLMs proliferate and evolve. Self-Harness removes the human engineer (and any stronger external agent) from the loop and lets the agent reshape the system around itself.

The method is an iterative loop with three stages: **Weakness Mining** (identify model-specific failure patterns from execution traces), **Harness Proposal** (generate diverse yet *minimal* modifications tied to those failures), and **Proposal Validation** (accept candidate edits only after regression testing). Starting from a minimal initial harness on Terminal-Bench-2.0, the approach yields consistent, model-specific gains across three base models from different families, turning concrete weaknesses into executable harness changes rather than generic instructions.

## Key points

- A fixed LLM diagnoses its own weaknesses, proposes minimal harness edits, and validates them via regression testing — no human engineer or stronger external agent required.
- Three-stage loop: Weakness Mining → Harness Proposal → Proposal Validation.
- Evaluated on Terminal-Bench-2.0 from a minimal seed harness with three base models: MiniMax M2.5, Qwen3.5-35B-A3B, and GLM-5.
- Held-out pass rates improved 40.5%→61.9%, 23.8%→38.1%, and 42.9%→57.1% across the three models.
- Produces *model-specific* harness configs: edits target each model's distinct failure modes rather than adding generic boilerplate.
- Positions agents as participants in reshaping their harnesses, not merely passive subjects of them.

## Concepts & entities

- [[agentic-harness-engineering]] — Self-Harness is an automated instance of this concept; see also [[agentic-harness-engineering-lin-2026]].
- [[code-as-harness]] — related view of the harness as engineered system.
- [[self-improving-agent]] — the agent improves its operating context (harness) rather than its weights.
- [[terminal-bench]] — evaluation benchmark used (Terminal-Bench-2.0).
- [[minimax-m2-series-minimax-2026]] — MiniMax M2.5 is one of the evaluated base models.

## References

- [arXiv:2606.09498](https://arxiv.org/abs/2606.09498) — "Self-Harness: Harnesses That Improve Themselves," Zhang et al., 2026 (Shanghai AI Laboratory).
- Local source text: `self-harness-harnesses-that-improve-themselves-zhang-2026.md`.
