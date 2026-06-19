---
title: "Agents' Last Exam (ALE)"
type: source
created: 2026-06-19
updated: 2026-06-19
sources:
  - agents-last-exam-sun-2026.md
arxiv_id: "2606.05405"
authors:
  - Yiyou Sun
  - et al.
first_author: Yiyou Sun
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Agents' Last Exam (ALE)

Surfaced via the alphaXiv weekly digest (2026-06-10).

## Summary

Agents' Last Exam (ALE) is an [[agent-benchmark]] led by [[uc-berkeley]] (first author Yiyou Sun; senior lead Dawn Song) that argues the gap between strong benchmark scores and real economic deployment is largely an *evaluation* problem: existing benchmarks rarely measure sustained performance on real, economically valuable workflows. ALE evaluates AI agents on long-horizon, economically valuable, real-world professional tasks with *verifiable* outcomes, built in collaboration with 250+ industry experts. Its taxonomy references the U.S. federal occupational standard (O*NET / SOC 2018), covering non-physical industries via 55 subfields grouped into 13 industry clusters, with 1K+ tasks (a 152-task public set spanning domains from CNC toolpath generation to single-cell clustering to chip signoff routing).

Tasks run in remote VM environments with a load/start/evaluate lifecycle that produces real artifacts scored against references (often a gate-and-score pattern; LLM judges only where genuinely needed). Results show ALE is far from saturated. The strongest configuration (Codex with GPT-5.5) — which scores 82% on Terminal-Bench — reaches only 23.3% overall pass on the harder CLI sub-benchmark, and on the hardest "Last-Exam" tier most agents score a 0% pass rate (average full-pass rate below 1% across mainstream configs). ALE is designed as a living benchmark whose task pool grows as new workflows are onboarded.

## Key points

- Premise: benchmark success has not translated to GDP-relevant impact because evaluations lack real, economically valuable, long-horizon workflows with verifiable outcomes.
- Built with 250+ industry experts; taxonomy grounded in O*NET / SOC 2018; 55 subfields, 13 industry clusters, 1K+ tasks (152-task public set).
- Tasks execute in remote VMs producing real artifacts (CAD models, toolpaths, reports, code); scored deterministically where possible via gate-and-score, LLM judges only when unavoidable.
- Three difficulty tiers: Near-Term (67 tasks, top pass rates ~40%), Full-Spectrum (55 tasks, one per subdomain), Last-Exam (38 hardest tasks, ~0% pass for most agents).
- Strongest config (Codex + GPT-5.5) hits only 23.3% overall on ALE-CLI vs 82% on Terminal-Bench; hardest tier far from saturated (avg full-pass rate below 1%).
- Positioned as a living benchmark and instrument for closing the benchmark-vs-economic-impact gap, not just another leaderboard.

## Concepts & entities

- [[agent-benchmark]] — ALE is a long-horizon agent evaluation benchmark
- [[uc-berkeley]] — leading institution
- Related: [[gaia]] — adjacent agent benchmark
- Related: economically valuable work, verifiable outcomes, long-horizon tasks (plain text — no dedicated pages)

## References

- [arXiv:2606.05405](https://arxiv.org/abs/2606.05405)
