---
title: Set-of-Mark
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - magma-yang-2025.md
status: draft
importance: medium
tags:
  - 2025
---

**Set-of-Mark (SoM)** is a visual-grounding technique used by [[magma]] and related GUI/vision systems. Clickable or otherwise interactable elements in an image are detected and overlaid with numbered marks before being shown to the model, turning element selection into a discrete label-prediction problem rather than raw pixel-coordinate regression.

- Overlays numbered marks on actionable UI / object elements
- Reduces grounding to label classification, which VLMs do well
- Used in [[magma]] for both pre-training and downstream action prediction
- Standard building block in modern [[gui-agent]] and [[vision-language-action-model]] stacks

Related: [[magma]], [[vision-language-action-model]], [[gui-agent]]

## References
- [[magma-yang-2025]]
