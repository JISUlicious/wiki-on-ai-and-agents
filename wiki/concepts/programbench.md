---
title: ProgramBench
type: concept
created: 2026-10-29
updated: 2026-10-29
sources:
  - agensh-scaling-organizational-intelligence-1024-agents-zhan-2026.md
  - gitswarm-decentralized-compounding-inference-shah-2026.md
  - harness-of-harness-multi-day-autonomous-development-yan-2026.md
  - kimi-k3-moonshot-2026.md
  - mimo-v2-6-scaling-reinforcement-learning-self-improvement-xiaomi-2026.md
status: draft
importance: medium
tags:
  - 2026
  - agent
---

# ProgramBench

**ProgramBench** is an agentic software-construction benchmark: the agent must rebuild a program **cleanroom**, from a compiled executable plus documentation, and is scored by hidden behavioural tests ([[harness-of-harness-multi-day-autonomous-development-yan-2026]]). [[agensh-scaling-organizational-intelligence-1024-agents-zhan-2026|Agensh]] describes it as program reconstruction from an execute-only reference. The full set has 200 tasks ([[gitswarm-decentralized-compounding-inference-shah-2026|GitSwarm]] uses a 50-of-200 subset). Hard tasks are real large codebases: Agensh's five hardest are FFmpeg, gromacs, pandoc, PHP-src and ctags. No original paper for the benchmark is ingested yet; everything here comes from papers that report on it.

It belongs with GameCraft-Bench and FrontierSWE as long-horizon, whole-program benchmarks (see [[agent-benchmark]]), as opposed to single-issue suites like [[swe-bench-verified]].

## Reported numbers

> [!warning] Not comparable across papers
> Each paper uses a different subset, metric, budget and harness. Read each row only against its own baseline; do not rank them against each other.

| Source | Subset / metric | Setting | Reported result |
|---|---|---|---|
| [[agensh-scaling-organizational-intelligence-1024-agents-zhan-2026\|Agensh]] | 5 hardest tasks; mean final test-pass rate | 6 h, no Internet, GPT-5.6-sol high; 1 → 128 agents | 19.31% → 28.78% (single runs) |
| [[gitswarm-decentralized-compounding-inference-shah-2026\|GitSwarm]] | Fixed 50-of-200 subset; mean score | Codex (GPT-5.5-high) workers, 100 / 500 worker-episodes | 73.1% / 79.4%, vs 63.1–65.1% for a "forced Codex" single agent |
| [[harness-of-harness-multi-day-autonomous-development-yan-2026\|Harness-of-Harness]] | Avg. test pass rate over hidden behavioural tests | Vanilla → HoH@3, three harness–model pairs | Codex + GPT-5.5 60.41 → 66.50; OpenCode + DeepSeek-V4-Pro 45.27 → 57.56; Pi + MiniMax-M3 35.83 → 52.68 |
| [[kimi-k3-moonshot-2026\|Kimi K3]] | Model-card table (metric not detailed on the page) | K3 at max effort | 77.8, vs 77.6 GPT-5.6 Sol, 76.8 Claude Fable 5 |
| [[mimo-v2-6-scaling-reinforcement-learning-self-improvement-xiaomi-2026\|MiMo-V2.6]] | Model-card table | MiMo-V2.6 Pro | 26.5, vs 37.0 for Claude Opus 5 |

The spread itself is the lesson: frontier models score in the high 70s in Kimi K3's table but in the 20s–30s in MiMo-V2.6's, which is only possible if the two reports use different subsets or scoring. The GitSwarm source page notes explicitly that its 79.4 cannot be compared with Agensh's 28.78%.

## What it is used to show

- **Multi-agent scaling.** Agensh (orchestrator-free, up to 1,024 agents) and GitSwarm (Git DAG with `informed_by=` references) use it to argue that many coordinated agents beat one. On GitSwarm, 94.7% of contributions are later built on. Agensh has no cost accounting; GitSwarm notes diminishing gains as input-token cost grows. See [[multi-agent]].
- **Harness iteration.** Harness-of-Harness counts ProgramBench pass rate in its 52.25% mean relative gain, but its budget-matched control was run only on GameCraft-Bench, so the ProgramBench gains are not budget-matched.
- **Frontier model cards.** Kimi K3 lists it among its best-in-suite results; MiMo-V2.6 lists it among benchmarks where it trails.

## Related

- [[agent-benchmark]] · [[multi-agent]] · [[agentic-harness-engineering]] · [[terminal-bench]] · [[swe-bench-verified]]
