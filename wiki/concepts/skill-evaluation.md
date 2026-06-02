---
title: Skill Evaluation
type: concept
created: 2026-06-02
updated: 2026-06-02
sources:
  - agentic-skills-in-the-wild-liu-2026.md
status: draft
importance: medium
tags:
  - 2026
---

# Skill Evaluation

**Skill evaluation** is the problem of measuring how well an [[agent-skills|agent skill]] actually helps — and doing so in *realistic* settings rather than with a hand-picked skill and a curated task. It is the measurement substrate that [[skill-optimization]] depends on (every text-space optimizer gates edits on a held-out skill score).

## Core issue

- **Idealized vs realistic gap.** [[agentic-skills-in-the-wild-liu-2026|Liu et al. 2026]] show that skill gains measured with a hand-selected skill **degrade sharply** when the agent must retrieve the right skill from a pool of ~34,000 real skills. The headline numbers in most skill papers assume the right skill is already in context — a generous assumption.
- **Retrieval is half the problem.** Realistic skill use is a retrieval task ([[agentic-search]]) before it is an instruction-following task. Query-specific refinement of the skill recovers much of the lost gain (Terminal-Bench 2.0: 57.7% → 65.5%).
- **Held-out validation as the optimizer signal.** [[skillopt-yang-2026|SkillOpt]] and [[skillmoo-gong-2026|SkillMOO]] both accept skill edits only on held-out improvement — making evaluation the inner loop of optimization, not just a final report.

## Relation to other concepts

- [[skill-optimization]] — uses evaluation as its acceptance gate
- [[agent-skills]] / [[skill-md-format]] — the artifact under test
- [[agent-benchmark]] — the broader benchmarking umbrella
- [[agentic-search]] — realistic skill use is a retrieval problem first
- [[longmemeval]] / [[swe-bench-live]] — adjacent realistic-setting benchmarks

## References

- [[agentic-skills-in-the-wild-liu-2026]] — the realistic skill-usage benchmark
