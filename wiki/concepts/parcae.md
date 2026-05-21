---
title: Parcae
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - parcae-scaling-laws-looped-2026.md
status: draft
importance: medium
tags:
  - 2026
---

**Parcae** is a stabilization recipe for looped language models proposed by Prairie et al. (2026). It treats the loop as a discrete-time linear time-invariant (LTI) system around its fixed point and parameterizes the recurrent update so the linearized dynamics have a **negative-diagonal A** matrix, guaranteeing contractive iteration and clean scaling behavior at depth.

- LTI-system framing of looped / [[recurrent-depth-transformer]] inference
- Negative-diagonal A enforces stable, contractive loop dynamics
- Enables clean scaling laws across loop count, width, and data
- Connects looped-LM theory with the empirical practice of [[openmythos]]-style training

Related: [[looped-transformer]], [[recurrent-depth-transformer]], [[scaling-laws]], [[openmythos]]

## References
- [[parcae-scaling-laws-looped-2026]]
