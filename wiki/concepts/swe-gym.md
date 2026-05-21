---
title: SWE-Gym
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - swe-gym-pan-2024.md
status: draft
importance: medium
tags:
  - 2024
  - benchmark
---

**SWE-Gym** is a training environment for software-engineering agents introduced by Pan et al. (2024). It collects 2,438 real Python tasks drawn from 11 GitHub repositories, each shipped with the original failing/passing unit tests so policies can be optimized against an executable reward signal.

- 2,438 Python tasks across 11 repos, packaged with their full test suites
- Designed for **RL training** of coding agents (unlike pure evaluation suites)
- Successor / training-time analog to the evaluation-focused [[swe-bench]]
- Bridges the gap between issue-fix prompting and verifiable, executable rollouts for [[agentic-rl]]

Related: [[swe-bench]], [[swe-bench-live]], [[llm-agent]], [[agentic-rl]]

## References
- [[swe-gym-pan-2024]]
