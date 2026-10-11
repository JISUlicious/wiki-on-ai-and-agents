---
title: "Dualformer: Controllable Fast and Slow Thinking by Learning with Randomized Reasoning Traces"
type: source
created: 2026-10-10
updated: 2026-10-10
sources:
  - dualformer-su-2024.md
arxiv_id: "2410.09918"
venue: "arXiv (ICLR 2025, general knowledge)"
authors:
  - DiJia Su
  - Sainbayar Sukhbaatar
  - Michael Rabbat
  - Yuandong Tian
first_author: DiJia Su
year: 2024
tags:
  - 2024
  - paper
status: complete
importance: medium
---

# Dualformer

**Source**: `sources/dualformer-su-2024.md` ([arXiv:2410.09918](https://arxiv.org/abs/2410.09918), 2024-10-13)
**Authors**: DiJia Su, Sainbayar Sukhbaatar, Michael Rabbat, Yuandong Tian, et al. ([[meta]] FAIR). Code: facebookresearch/dualformer.

## Summary

**A single Transformer that runs in fast mode (solution only), slow mode (reasoning trace + solution), or auto mode (choosing for itself).** It is trained on **randomized reasoning traces**: parts of A*-search traces are strategically dropped during training, so one model learns both modes.

## Results (30×30 mazes; math generalization)

| Mode | Result |
|---|---|
| Slow | **97.6%** optimal, against Searchformer's 93.3%, with **45.5% fewer** reasoning steps |
| Fast | **80%** optimal, against 30% for a solution-only model |
| Auto | **96.6%** optimal, with 59.9% fewer steps than Searchformer |

The same trace-dropping technique also improved LLM fine-tuning on math.

## Why it matters

- **Fast and slow are learnable modes of one network, not two architectures.** That echoes the dual-process critique that speed alone doesn't tell you which "system" is in use.
- **An early form of adaptive reasoning.** The model decides how much to think, which reasoning-effort controls now offer ([[reasoning-effort]], [[test-time-compute]]).

## Concepts & entities

[[dual-process-theory]] · [[test-time-compute]] · [[reasoning-effort]] · [[planning]] · [[system-1x-saha-2024]]
