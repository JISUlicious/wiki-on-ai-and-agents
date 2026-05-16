---
title: Offline Consolidation
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - sleep-time-compute-lin-2025.md
status: draft
importance: medium
tags:
  - 2025
---

Offline consolidation is the practice of doing compute *between* user queries — during idle or "sleep" time — to pre-derive useful state that's then cheaply available when the next query arrives. Rather than recomputing summaries, inferences, or memory rewrites on every turn, the agent runs background passes that distill context into a more compact, more queryable form. The metaphor is borrowed from biological memory consolidation during sleep, where the hippocampus replays and reorganizes the day's experiences. Introduced by [[sleep-time-compute]] (Lin/Snell, Letta 2025), which reports ~5× test-time compute reduction and ~+18% accuracy from moving work to idle time. The same pattern shows up as the "auto-dream" background memory consolidator in Claude Code.

## Where it appears

- [[sleep-time-compute]] — Direct introduction; quantifies the compute/accuracy tradeoff.
- [[memory-consolidation]] — The general biological/cognitive analog.
- [[memory-management]] — Offline consolidation is one phase in the lifecycle of structured agent memory.

## References

- [[sleep-time-compute-lin-2025]] — Sleep-Time Compute: Beyond Inference Scaling at Test-Time (Lin, Snell et al., Letta 2025).
