---
title: Trace-of-Mark
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

**Trace-of-Mark (ToM)** is [[magma]]'s motion-planning counterpart to [[set-of-mark]]. Trajectories from videos (e.g., a hand reaching for an object, a cursor moving to a button) are encoded as a sequence of marked points across frames, giving the model a discrete supervisory signal for *how things move* in addition to *what they are*.

- Encodes trajectories as marked traces across frames
- Provides motion supervision for [[vision-language-action-model]] pre-training
- Complements [[set-of-mark]]: SoM grounds *what to act on*, ToM grounds *how to move*
- Bridges GUI navigation and physical [[embodied-agent]] control in one objective

Related: [[magma]], [[vision-language-action-model]], [[embodied-agent]], [[set-of-mark]]

## References
- [[magma-yang-2025]]
