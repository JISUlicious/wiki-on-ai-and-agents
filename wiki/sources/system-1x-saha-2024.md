---
title: "System-1.x: Learning to Balance Fast and Slow Planning with Language Models"
type: source
created: 2026-10-10
updated: 2026-10-10
sources:
  - system-1x-saha-2024.md
arxiv_id: "2407.14414"
venue: "arXiv (ICLR 2025, general knowledge)"
authors:
  - Swarnadeep Saha
  - Archiki Prasad
  - Justin Chih-Yao Chen
  - Peter Hase
first_author: Swarnadeep Saha
year: 2024
tags:
  - 2024
  - paper
status: complete
importance: medium
---

# System-1.x

**Source**: `sources/system-1x-saha-2024.md` ([arXiv:2407.14414](https://arxiv.org/abs/2407.14414), 2024-07-19)
**Authors**: Swarnadeep Saha, Archiki Prasad, Justin Chih-Yao Chen, Peter Hase, et al.

## Summary

Language models can plan in two modes:
- **System-1**: generate the plan directly, with no search.
- **System-2**: plan step by step with explicit search. This is usually better, but expensive and infeasible for long plans.

**The System-1.x Planner** has three parts:
- a **controller**, which splits the problem into sub-goals and labels each easy or hard;
- a **System-1 planner**, for the easy sub-goals;
- a **System-2 planner**, for the hard ones.

A user-set **hybridization factor x** controls the mix: "System-1.75" means more search, "System-1.5" less. All three components are fine-tuned on one LLM from search traces. A neuro-symbolic variant uses a symbolic A* planner as System 2.

## Results

On Maze Navigation and Blocksworld under a state-exploration budget, System-1.x outperforms:
- a System-1 planner;
- a System-2 planner trained to imitate A*;
- symbolic A* itself.

It is controllable through x, and robust to the search algorithm used for training (BFS, DFS, A*).

## Why it matters

- **System 1 vs System 2 as a dial, with a router deciding which sub-problem gets which mode.** That is the **architectural** reading of dual process, and the one closest to how [[system-one-model|System One models]] are pitched: a cheap fast component handles the easy decisions, and the expensive deliberate one handles the rest.
- **The controller itself is the interesting part.** Deciding easy vs hard is a classification, the kind of decision a typed decision model is built for. See [[system-one-models-and-dual-process-thinking]].

## Concepts & entities

[[dual-process-theory]] · [[planning]] · [[test-time-compute]] · [[dualformer-su-2024]] · [[system-one-model]]
