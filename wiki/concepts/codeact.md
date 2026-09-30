---
title: CodeAct
type: concept
created: 2026-06-19
updated: 2026-09-30
sources:
  - codeact-wang-2024.md
status: draft
importance: medium
tags:
  - 2026
---

# CodeAct

CodeAct is an agent action paradigm that uses executable Python code as the agent's action space, replacing fixed JSON tool-call formats. Expressing actions as code lets a single action compose control flow, reuse libraries, and chain operations, which improves task success and reduces the number of steps. It exemplifies the [[code-as-harness]] approach.

## References

- [[codeact-wang-2024]]

## Related 2026 sources

- [[is-bash-all-you-need-enterprise-agents-mak-2026]] — Microsoft: bash alone beats typed tools end-to-end (+5–25 pts, 19–72% fewer tokens); programmatic tool calling mixed
