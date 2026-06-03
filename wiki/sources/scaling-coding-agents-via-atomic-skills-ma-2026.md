---
title: "Scaling Coding Agents via Atomic Skills"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - scaling-coding-agents-via-atomic-skills-ma-2026.md
arxiv_id: "2604.05013"
authors:
  - Yingwei Ma et al.
first_author: Yingwei Ma
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Scaling Coding Agents via Atomic Skills

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

This paper (HKUST, NUS, PKU, SJTU, BUPT) argues that LLM coding agents trained on composite benchmarks (e.g. bug-fixing) overfit to specific task formats and generalize poorly. It proposes a scaling paradigm that shifts from task-level optimization to **atomic skill mastery**. The authors formalize five fundamental atomic skills — code localization, code editing, unit-test generation, issue reproduction, and code review — treated as "basis vectors" that compose into complex software-engineering tasks. These atomic skills are claimed to be more generalizable and composable than composite tasks.

The training approach performs **joint reinforcement learning** across the five atomic skills simultaneously, so they improve together without negative interference or trade-offs between them. Crucially, improvements on atomic skills generalize to *unseen* composite tasks — bug-fixing, code refactoring, ML engineering, and code security — supporting the thesis that mastering atomic primitives is a better scaling axis than scaling task-specific training. Joint RL raises average performance by 18.7% across the five atomic skills and five composite tasks.

## Key points

- Critiques composite-benchmark training (e.g. SWE-bench-style bug-fixing) for task-specific overfitting and weak generalization.
- Defines five atomic skills as basis vectors: code localization, code editing, unit-test generation, issue reproduction, code review.
- Joint RL over all five skills improves them consistently without negative interference or inter-skill trade-offs.
- Atomic-skill gains transfer to unseen composite tasks: bug-fixing, refactoring, ML engineering, code security.
- Reframes the scaling axis for coding agents from task-level optimization to composable skill mastery.
- Joint RL yields +18.7% average across 5 atomic skills and 5 composite tasks.

## Concepts & entities

[[agent-skills]] · [[skill-acquisition]] · [[skill-library]] · [[code-as-harness]] · [[reinforcement-learning]] · [[agentic-rl]] · [[llm-agent]] · [[swe-bench]] · [[humaneval]]

## References

- arXiv: [arXiv:2604.05013](https://arxiv.org/abs/2604.05013)
