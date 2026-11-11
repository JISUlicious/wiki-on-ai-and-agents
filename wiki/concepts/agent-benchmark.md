---
title: Agent Benchmark
type: concept
created: 2026-05-21
updated: 2026-11-10
sources:
  - gaia-mialon-2023.md
  - swe-bench-jimenez-2023.md
  - terminal-bench-benchmarking-agents-cli-merrill-2026.md
  - alphaeval-evaluating-agents-in-production-lu-2026.md
  - stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026.md
  - measuring-ai-agents-as-teammates-collaborative-intelligence-wu-2026.md
  - lifeval-personalized-agentic-consumer-tasks-yang-2026.md
status: draft
importance: medium
---

# Agent Benchmark

Agent benchmarks are evaluation suites designed to measure LLM-based **agents** rather than raw next-token models. They typically require multi-step interaction with tools, environments, or codebases — [[tool-use|tool use]], browser/GUI navigation, software engineering on real repos, long-horizon planning, memory over many turns, etc. — and score end-to-end task completion rather than per-token accuracy.

- Distinguishing feature vs. static QA: the agent must **act**, observe results, and continue.
- Coverage spans general assistant tasks, software engineering, GUI/computer use, and long-context memory.
- Reliability and contamination resistance are recurring concerns (see [[swe-bench-live]] for a "live" variant).
- Often paired with a reference scaffold so the benchmark also implicitly evaluates the harness.

## What the suites measure

| Benchmark | Scope | Reported result |
|---|---|---|
| [[gaia]] | 466 tool-using assistant questions | Humans 92%, GPT-4 with plugins about 15% at release |
| [[swe-bench]] | 2,294 real GitHub issues | Under 4% at release; [[swe-bench-verified\|Verified]] now reported around 70–80% |
| [[terminal-bench]] | 89 hard command-line tasks | Under 65%; top is Codex CLI + GPT-5.2 at 63% |
| [[alphaeval-evaluating-agents-in-production-lu-2026\|AlphaEval]] | 94 production tasks from 7 companies | Best 64.41 of 100 |

These figures come from different dates and scoring rules, so they can't rank the suites against each other.

## What the score actually measures

An agent benchmark scores a model and harness together. AlphaEval puts the same Opus 4.6 at 64.41 through Claude Code and 53.45 through Codex. [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026|Zhang et al.]] propose either locking the harness or varying it as a controlled factor. Terminal-Bench lists model and scaffold pairs explicitly for this reason. See [[harness-vs-model-attribution]].

For how runs are graded, see [[agent-evaluation]].

## Related

- [[benchmark]] — hub of every benchmark page in this wiki
- [[gaia]] — general assistant benchmark
- [[swe-bench]] — repo-level software engineering
- [[swe-bench-live]] — contamination-resistant variant
- [[osworld]] — desktop GUI / computer use
- [[longmemeval]] — long-horizon memory

## References

- [[gaia-mialon-2023]]
- [[swe-bench-jimenez-2023]]
- [[terminal-bench-benchmarking-agents-cli-merrill-2026]]
- [[alphaeval-evaluating-agents-in-production-lu-2026]]
- [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]] for full grouping.

- [[alphaeval-evaluating-agents-in-production-lu-2026]] — AlphaEval Evaluating Agents in Production
- [[arc-agi-3-arc-prize-foundation-2026]] — ARC AGI 3 A New Challenge for Frontier
- [[autolab-long-horizon-auto-research-xu-2026]] — AutoLab Can Frontier Models Solve Long Horizon Auto
- [[building-verifiers-for-computer-use-agents-rosset-2026]] — The Art of Building Verifiers for Computer Use
- [[screening-is-enough-nakanishi-2026]] — Screening Is Enough
- [[stanford-edgar-filings-dataset-bettencourt-2026]] — SEFD-v1 SEC-filings dataset + EDGAR-Forecast/OCR benchmarks
- [[reliability-without-validity-llm-as-a-judge-norman-2026]] — largest LLM-as-a-Judge audit; exact-match overstates quality
- [[naturebench-coding-agents-nature-family-papers-wang-2026]] — NatureBench: Nature-family discovery tasks for coding agents
- [[paper-assistant-tool-jayaram-2026]] — Google Paper Assistant Tool: agentic scientific review/verification at scale

## Newer evidence (2026-11-10 sweep)

- Collaborative Intelligence Benchmark: 240 teamwork tasks, scoring each model only on tasks it can solve alone; best 82.2. ([[measuring-ai-agents-as-teammates-collaborative-intelligence-wu-2026]])
- LifeVal: 169 personalized consumer tasks weighted by a 3,837-person willingness-to-pay survey. ([[lifeval-personalized-agentic-consumer-tasks-yang-2026]])
