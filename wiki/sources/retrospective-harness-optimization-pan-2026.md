---
title: "Retrospective Harness Optimization: Improving LLM Agents via Self-Preference over Trajectory Rollouts"
type: source
created: 2026-06-09
updated: 2026-06-09
sources:
  - retrospective-harness-optimization-pan-2026.md
arxiv_id: "2606.05922"
authors:
  - Wenbo Pan
  - et al.
first_author: Wenbo Pan
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Retrospective Harness Optimization: Improving LLM Agents via Self-Preference over Trajectory Rollouts

Surfaced while researching [[agentic-harness-engineering]].

[arXiv:2606.05922](https://arxiv.org/abs/2606.05922) — Wenbo Pan et al. (City University of Hong Kong; Microsoft Research Asia), June 2026.

## Summary

Agents rely on a harness of skills, tools, and workflows that must be continually improved to adapt to new tasks, but existing harness-optimization methods need ground-truth validation sets — labeled data that is hard to obtain in deployment. **Retrospective Harness Optimization (RHO)** is a self-supervised alternative that improves the harness using only *past trajectories*, with no external grading. It selects a diverse coreset of challenging tasks from prior runs, re-solves them in parallel, and analyzes the rollouts via self-validation (within a trajectory) and self-consistency (disagreement across parallel trajectories) to localize failure modes.

From that analysis the agent proposes N candidate harness updates and selects the most effective one through its own *pairwise self-preference*. Across three domains — software engineering, technical work, and knowledge work — a single optimization round lifts SWE-Bench Pro pass rate from 59% to 78% without any external grader. The analysis shows RHO targets prior failure modes, changes the agent's behavior patterns, and sustains higher accuracy over long-horizon sessions.

## Key points

- Problem: harness optimization usually requires ground-truth validation sets that deployments rarely have; RHO removes that requirement.
- Self-supervised loop over *past trajectories*: coreset selection → parallel re-solve → self-validation + self-consistency → candidate harness updates → pairwise self-preference selection.
- No external grading needed at any stage; the agent judges its own rollouts.
- Single round raises SWE-Bench Pro from 59% → 78% pass rate.
- Evaluated across software engineering, technical work, and knowledge work domains.
- Optimized harness demonstrably targets prior failure modes and sustains accuracy in long-horizon sessions.

## Concepts & entities

- [[agentic-harness-engineering]] — RHO optimizes the agent's harness (skills, tools, workflows) directly.
- [[self-improving-agent]] — self-supervised improvement from the agent's own past trajectories.
- [[skill-optimization]] — harness updates include refining the agent's skill set.
- [[llm-agent]] — the system being optimized.
- [[dynamic-workflows]] — workflows are part of the harness RHO mutates.

## References

- [[agentic-harness-engineering]] — primary concept this source backs.
- [[autoharness-lou-2026]] — related automatic-harness-improvement work.
- [[meta-harness-lee-2026]] — adjacent harness-optimization line.
