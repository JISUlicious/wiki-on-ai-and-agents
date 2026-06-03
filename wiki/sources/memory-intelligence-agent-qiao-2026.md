---
title: "Memory Intelligence Agent"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - memory-intelligence-agent-qiao-2026.md
arxiv_id: "2604.04503"
authors:
  - Jingyang Qiao et al.
first_author: Jingyang Qiao
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Memory Intelligence Agent

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

The Memory Intelligence Agent (MIA) framework (East China Normal University and collaborators) targets memory for **deep research agents** (DRAs) — LLMs that interleave reasoning with external tools. Prior methods retrieve similar past trajectories to aid reasoning but suffer from ineffective memory evolution and growing storage/retrieval cost. MIA uses a **Manager-Planner-Executor** architecture: a non-parametric Memory Manager that stores compressed historical search trajectories; a parametric Planner agent that produces search plans; and an Executor agent that searches and analyzes information under the plan's guidance.

Training uses an **alternating reinforcement learning** scheme to improve Planner-Executor cooperation, plus **test-time learning** where the Planner evolves on-the-fly during inference without interrupting reasoning. A bidirectional conversion loop links parametric and non-parametric memory for efficient evolution, and reflection plus unsupervised judgment mechanisms support open-world self-evolution. Across eleven benchmarks MIA boosts SOTA models (e.g. +9% / +6% for GPT-5.4 on LiveVQA / HotpotQA) and, with a lightweight Qwen2.5-VL-7B executor, averages +31%, beating the far larger Qwen2.5-VL-32B by 18%. Unsupervised MIA nearly matches its supervised counterpart and self-improves across training iterations.

## Key points

- Manager-Planner-Executor design: non-parametric memory store + parametric planning agent + searching/analysis executor.
- Memory Manager compresses historical search trajectories rather than storing raw traces.
- Alternating RL trains Planner and Executor to cooperate; both synergistically optimize strategies.
- Planner does **test-time learning**, updating on-the-fly during inference without halting reasoning.
- Bidirectional parametric ↔ non-parametric memory conversion loop enables efficient memory evolution.
- Reflection + unsupervised judgment support open-world self-evolution; unsupervised ≈ supervised performance.
- Results across 11 benchmarks: +9%/+6% on GPT-5.4 (LiveVQA/HotpotQA); 7B executor +31% avg, beating 32B by 18%.
- Notes long-context memory baselines struggle with multi-turn tool interaction, where MIA outperforms.

## Concepts & entities

[[memory-management]] · [[memory-evolution]] · [[test-time-memory]] · [[multi-agent-memory]] · [[llm-agent]] · [[reinforcement-learning]] · [[agentic-search]] · [[reflection-mechanism]] · [[multimodal-llm]]

## References

- arXiv: [arXiv:2604.04503](https://arxiv.org/abs/2604.04503)
- Code: https://github.com/ECNU-SII/MIA
