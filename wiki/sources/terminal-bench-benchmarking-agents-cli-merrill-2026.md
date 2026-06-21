---
title: "Terminal-Bench: Benchmarking Agents on Hard, Realistic Tasks in Command Line Interfaces"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - terminal-bench-benchmarking-agents-cli-merrill-2026.md
arxiv_id: "2601.11868"
authors:
  - Mike A. Merrill
  - Alexander G. Shaw
  - et al.
first_author: Mike A. Merrill
year: 2026
venue: arXiv (cs.SE)
tags: [2026]
status: complete
importance: high
---

# Terminal-Bench: Benchmarking Agents on Hard, Realistic Tasks in Command Line Interfaces

> Surfaced while researching harness-vs-model performance attribution.

## Summary

This is the canonical source for the [[terminal-bench]] benchmark (Merrill, Shaw, et al., 2026; Stanford / Laude Institute / Anthropic and a large multi-institution author list). Terminal-Bench is a framework for evaluating [[llm-agent]]s on realistic, hard work in command-line environments — the terminal being the standard interface for software engineering, scientific computing, cybersecurity, and ML. The released **Terminal-Bench 2.0** dataset is 89 carefully curated tasks, each with a unique containerized (Docker) environment, a human-written reference solution, and outcome-based tests that check the final container state rather than the agent's commands.

Crucially, the benchmark is harness-aware: tasks use the **Harbor** format/harness, which runs many agent scaffolds (Claude Code, Codex CLI, OpenHands, Mini-SWE-Agent) plus the authors' own neutral testbed scaffold **Terminus 2**. The leaderboard therefore reports **(agent-harness × model)** pairs, making scaffold quality visible rather than conflated into a single model number — directly addressing the misattribution concern of [[agentic-harness-engineering]].

## Key points

- **Dataset:** 89 hard tasks, each manually verified by three human reviewers; tasks span software engineering (largest category) plus non-engineering categories like "personal assistant" and "video processing"; outcome-driven (tests inspect final container state).
- **Frontier is far from saturated:** frontier models/agents resolve **< 65%** of tasks; smaller models around **15%**.
- **Top of leaderboard:** **Codex CLI + GPT-5.2** is highest at **63%** average resolution, followed by Terminus 2 with Claude Opus 4.5 / Gemini 3 Pro.
- **Harness matters and is exposed:** the leaderboard lists model+scaffold pairs explicitly (e.g., "GPT-5.2 (Codex CLI)", "Claude Opus 4.5 (Terminus 2)"); best open-weight result is Terminus 2 + Kimi K2 Thinking at ~36%.
- **Terminus 2** is introduced as a neutral scaffold so models can be compared on equal footing; closed-source agent pairings occupy the top 13 rankings.
- **Error analysis:** a failure-mode taxonomy (adapted from MAST) finds execution errors dominate, with coherence and verification failures at comparable rates. Released at tbench.ai.

## Concepts & entities

- [[terminal-bench]] — the benchmark this paper defines (canonical source)
- [[agent-benchmark]] · [[llm-agent]] · [[code-generation]]
- [[agentic-harness-engineering]] — the (harness × model) leaderboard design makes scaffold quality measurable
- Related: [[swe-bench]] · [[code-as-harness]]

## References

- [arXiv:2601.11868](https://arxiv.org/abs/2601.11868) — "Terminal-Bench: Benchmarking Agents on Hard, Realistic Tasks in Command Line Interfaces", Merrill, Shaw, et al., 2026
- Harness-attribution argument that cites Terminal-Bench: [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]]
