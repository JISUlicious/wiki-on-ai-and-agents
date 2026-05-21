---
title: Parameter Sharing
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - looped-transformers-learning-algorithms-yang-2023.md
status: draft
importance: medium
tags:
  - architecture
---

**Parameter sharing** is the architectural choice of reusing the *same* weights across multiple positions, layers, or timesteps rather than allocating fresh parameters per location. It trades raw expressivity for parameter efficiency, inductive bias toward iterative computation, and improved [[length-generalization]]. [[looped-transformer]], [[universal-transformer]], and [[recurrent-depth-transformer]] are all instances; classical RNNs share weights across time by definition.

- Reduces parameter count; increases effective compute per parameter
- Encodes an inductive bias for *iterative* algorithmic computation
- Pure tying loses some expressivity (see [[timestep-encoding]] for a fix)
- Foundational to looped / universal / recurrent-depth families

Related: [[looped-transformer]], [[universal-transformer]], [[recurrent-depth-transformer]], [[timestep-encoding]], [[length-generalization]]

## References
- [[looped-transformers-learning-algorithms-yang-2023]]
