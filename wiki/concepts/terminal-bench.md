---
title: Terminal-Bench
type: concept
created: 2026-06-19
updated: 2026-06-22
sources:
  - terminal-bench-benchmarking-agents-cli-merrill-2026.md
  - agentic-harness-engineering-lin-2026.md
  - self-harness-harnesses-that-improve-themselves-zhang-2026.md
  - meta-harness-lee-2026.md
  - natural-language-agent-harnesses-pan-2026.md
status: complete
importance: medium
tags:
  - 2026
---

# Terminal-Bench

**Terminal-Bench** ([[terminal-bench-benchmarking-agents-cli-merrill-2026|Merrill et al. 2026]], Stanford / Laude Institute / Anthropic) is an agentic benchmark that evaluates [[llm-agent|LLM agents]] on **terminal/shell tasks** — driving a real command line to accomplish software and systems goals, scored by verifiable outcomes. **Terminal-Bench-2.0** has **89 hard CLI tasks** that no model fully resolves (frontier <65%, small models ~15%). Critically, the leaderboard is a **(harness × model) grid** — each entry pairs a scaffold with a model — with **Terminus 2** as the neutral reference scaffold and **Harbor** as the execution harness. This design makes it the de-facto proving ground for **[[agentic-harness-engineering|harness engineering]]**: with the base model held fixed and only the scaffold changing, Terminal-Bench deltas isolate the contribution of the harness (see [[harness-vs-model-attribution]]).

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

- [[terminal-bench-benchmarking-agents-cli-merrill-2026]] — the benchmark's defining paper.
- [[agentic-harness-engineering-lin-2026]] · [[self-harness-harnesses-that-improve-themselves-zhang-2026]] · [[meta-harness-lee-2026]] · [[natural-language-agent-harnesses-pan-2026]]
- [[harness-vs-model-attribution]] — using Terminal-Bench's grid to separate scaffold from model.
