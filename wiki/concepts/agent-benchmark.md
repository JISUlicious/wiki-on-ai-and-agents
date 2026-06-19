---
title: Agent Benchmark
type: concept
created: 2026-05-21
updated: 2026-05-21
sources: []
status: draft
importance: medium
tags: []
---

# Agent Benchmark

Agent benchmarks are evaluation suites designed to measure LLM-based **agents** rather than raw next-token models. They typically require multi-step interaction with tools, environments, or codebases — tool use, browser/GUI navigation, software engineering on real repos, long-horizon planning, memory over many turns, etc. — and score end-to-end task completion rather than per-token accuracy.

- Distinguishing feature vs. static QA: the agent must **act**, observe results, and continue.
- Coverage spans general assistant tasks, software engineering, GUI/computer use, and long-context memory.
- Reliability and contamination resistance are recurring concerns (see [[swe-bench-live]] for a "live" variant).
- Often paired with a reference scaffold so the benchmark also implicitly evaluates the harness.

## Related

- [[gaia]] — general assistant benchmark
- [[swe-bench]] — repo-level software engineering
- [[swe-bench-live]] — contamination-resistant variant
- [[osworld]] — desktop GUI / computer use
- [[longmemeval]] — long-horizon memory

## References

- [[gaia]]
- [[swe-bench]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]] for full grouping.

- [[alphaeval-evaluating-agents-in-production-lu-2026]] — AlphaEval Evaluating Agents in Production
- [[arc-agi-3-arc-prize-foundation-2026]] — ARC AGI 3 A New Challenge for Frontier
- [[autolab-long-horizon-auto-research-xu-2026]] — AutoLab Can Frontier Models Solve Long Horizon Auto
- [[building-verifiers-for-computer-use-agents-rosset-2026]] — The Art of Building Verifiers for Computer Use
- [[screening-is-enough-nakanishi-2026]] — Screening Is Enough
