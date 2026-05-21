---
title: SWE-bench
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - swe-bench-jimenez-2023.md
status: draft
importance: high
tags:
  - 2023
  - benchmark
---

**SWE-bench** is the foundational coding-agent benchmark introduced by Jimenez et al. (2023). It contains 2,294 real GitHub issues drawn from 12 popular Python repositories, paired with the corresponding fix commits and passing unit tests, so an agent's patch can be graded by executing the test suite.

- 2,294 Python repository issues with ground-truth fix patches and tests
- Evaluation-only: agents are scored by test pass rate after patch application
- The umbrella concept that [[swe-bench-live]] (freshness) and [[swe-gym]] (training) extend
- Has become the canonical headline benchmark for [[llm-agent]] coding ability

Related: [[swe-bench-live]], [[swe-gym]], [[llm-agent]], [[agentic-rl]]

## References
- [[swe-bench-jimenez-2023]]
