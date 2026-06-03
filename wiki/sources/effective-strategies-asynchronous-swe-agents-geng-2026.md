---
title: "Effective Strategies for Asynchronous Software Engineering Agents"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - effective-strategies-asynchronous-swe-agents-geng-2026.md
arxiv_id: "2603.21489"
authors:
  - Jiayi Geng
  - Graham Neubig
first_author: Jiayi Geng
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Effective Strategies for Asynchronous Software Engineering Agents

Surfaced via the alphaXiv / NLP-newsletter digest.

[arXiv:2603.21489](https://arxiv.org/abs/2603.21489) — Geng & Neubig (CMU LTI), 2026.

## Summary

The paper studies how to make multiple SWE agents collaborate asynchronously on long-horizon, multi-subtask software projects, where naive parallelism fails because concurrent edits interfere, dependencies are hard to synchronize, and partial progress is hard to merge. Borrowing from how human developers coordinate on large codebases, the authors introduce **CAID (Centralized Asynchronous Isolated Delegation)**, a coordination paradigm built on three SWE primitives: centralized task delegation, asynchronous execution, and isolated workspaces.

A central **manager** builds a dependency-aware task plan and delegates subtasks to **engineer** agents, each working concurrently in its own `git worktree`; progress is consolidated through structured integration with executable, test-based verification. The key empirical finding is that **branch-and-merge** is the central coordination mechanism for reliable multi-agent collaboration, and that ordinary version-control primitives (`git worktree`, `git commit`, `git merge`) plus test suites make it executable and reliable.

## Key points

- CAID improves accuracy over single-agent baselines by **26.7% absolute on PaperBench** (paper reproduction) and **14.3% on Commit0** (Python library development from scratch).
- Three core primitives: centralized delegation, asynchronous execution, isolated workspaces — each engineer edits files only within its own `git worktree`.
- Branch-and-merge identified as the pivotal coordination mechanism; shared/protected files engineers may not commit to, conflicts resolved by pulling latest `main` into the worktree.
- Worktrees periodically sync to the latest integrated state and are deleted after completion; verification runs concurrently inside each worktree.
- Evaluated on Commit0-Lite and PaperBench Code-Dev with multiple backbones (e.g., MiniMax 2.5); single-agent uses `max_iterations=100`, multi-agent uses 50 (manager) / 80 (engineer).
- Maps human software-collaboration infrastructure onto agent coordination rather than inventing a bespoke protocol.

## Concepts & entities

- [[multi-agent-memory|multi-agent]] — CAID is a structured multi-agent coordination paradigm.
- [[code-as-harness]] — Uses git worktree/commit/merge as the executable coordination harness.
- [[swe-bench]] — Adjacent SWE-agent evaluation context; CAID instead uses PaperBench and Commit0.
- [[llm-agent]] — Manager + engineer agents are LLM agents.
- [[graham-neubig]] — Co-author (CMU LTI).

## References

- PaperBench [Starace et al., 2025] — paper-reproduction benchmark.
- Commit0 [Zhao et al., 2024] — from-scratch Python library implementation benchmark.
- Project code: https://github.com/JiayiGeng/CAID
