---
title: "Ralph Wiggum as a 'software engineer'"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - ralph-wiggum-software-engineer-huntley-2025.html
  - ralph-loop-plugin-readme-2026-11-02.md
authors:
  - Geoffrey Huntley
year: 2025
url: https://ghuntley.com/ralph/
introduces:
  - [[ralph-loop]]
status: complete
importance: medium
tags:
  - 2025
---

# Ralph Wiggum as a "software engineer"

Geoffrey Huntley, [ghuntley.com/ralph](https://ghuntley.com/ralph/), 14 July 2025. The later plugin README is included here because it defines Anthropic's adaptation.

## Summary

Huntley presents [[ralph-loop|Ralph]] as a technique that, in its purest form, is a Bash loop feeding a prompt file to a coding agent again and again: `while :; do cat PROMPT.md | claude-code ; done`. The loop has no built-in exit condition.

## Key points

- **The claim.** Huntley says Ralph can replace most outsourcing for greenfield projects, while admitting it has defects. This is his opinion, not a measurement.
- **Anthropic's adaptation.** The `ralph-loop` plugin keeps the loop inside one session. A Stop hook blocks exit and feeds the same prompt back. Completion is an agreed phrase the model prints, with `--max-iterations` as the safety limit.
- **Why it matters.** That phrase is a form of self-report, and the plain loop has no completion check of its own. See [[false-success]] and [[goal-command-lineage-repos-2026]].

## Concepts & entities

- [[ralph-loop]] · [[claude-code]] · [[harness-extension-and-goal-loop-axes]]

## References

- [Huntley's post](https://ghuntley.com/ralph/)
- [`ralph-loop` plugin](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/ralph-loop)
