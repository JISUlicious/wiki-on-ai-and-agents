---
title: Terminal-Bench
type: concept
created: 2026-06-19
updated: 2026-06-19
sources:
  - agentic-harness-engineering-lin-2026.md
  - self-harness-harnesses-that-improve-themselves-zhang-2026.md
  - meta-harness-lee-2026.md
  - natural-language-agent-harnesses-pan-2026.md
status: draft
importance: medium
tags:
  - 2026
---

# Terminal-Bench

**Terminal-Bench** is an agentic benchmark that evaluates [[llm-agent|LLM agents]] on **terminal/shell tasks** — driving a real command line to accomplish software and systems goals, scored by verifiable outcomes. By 2026 it (and its v2, **Terminal-Bench-2.0**) had become the de-facto proving ground for **[[agentic-harness-engineering|harness engineering]]**: because the base model is held fixed and only the scaffold changes, Terminal-Bench deltas isolate the contribution of the harness.

## Why it recurs in harness research

Nearly every harness-optimization paper in this wiki reports Terminal-Bench numbers, which makes it the shared yardstick for the field:

- [[agentic-harness-engineering-lin-2026|AHE]] — lifts pass@1 from a bash-only seed of 69.7% to 77.0% on Terminal-Bench 2, beating human-designed Codex.
- [[self-harness-harnesses-that-improve-themselves-zhang-2026|Self-Harness]] — a fixed LLM self-improves its harness (e.g. 40.5→61.9% held-out) across MiniMax-M2.5 / Qwen3.5 / GLM-5.
- [[meta-harness-lee-2026|Meta-Harness]] and [[natural-language-agent-harnesses-pan-2026]] also evaluate here.

Because gains transfer (frozen evolved harnesses also help on [[swe-bench|SWE-bench]]), Terminal-Bench results are read as evidence about harness *structure*, not just leaderboard position.

## Related

- [[agentic-harness-engineering]] — the research area Terminal-Bench primarily serves.
- [[agent-benchmark]] — the broader category; cf. [[swe-bench]], [[gaia]], and ALE ([[agents-last-exam-sun-2026|Agents' Last Exam]]).
- [[code-as-harness]] · [[llm-agent]]

## References

- [[agentic-harness-engineering-lin-2026]] · [[self-harness-harnesses-that-improve-themselves-zhang-2026]] · [[meta-harness-lee-2026]] · [[natural-language-agent-harnesses-pan-2026]]
