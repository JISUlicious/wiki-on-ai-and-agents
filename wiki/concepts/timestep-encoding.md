---
title: Timestep Encoding
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - expressive-power-looped-transformers-xu-2024.md
status: draft
importance: medium
tags:
  - 2024
---

**Timestep encoding** is Xu & Sato's (2024) fix for the expressivity gap that arises when a [[looped-transformer]] ties all its weights across iterations. A per-loop scaling vector η(t), produced from a sinusoidal time embedding passed through a small shared MLP, is multiplied into the block's activations — letting the same weights behave differently at different loop indices without giving up parameter sharing.

- Sinusoidal embedding of loop index t → small shared MLP → scaling vector η(t)
- Restores universal-approximation-class expressivity that pure weight tying loses
- Cheap addition: only one small MLP regardless of loop depth
- Conceptual cousin of diffusion-model timestep conditioning

Related: [[looped-transformer]], [[expressive-power-looped-transformers-xu-2024]], [[parameter-sharing]]

## References
- [[expressive-power-looped-transformers-xu-2024]]
