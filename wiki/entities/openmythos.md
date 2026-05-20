---
title: OpenMythos
type: entity
created: 2026-05-20
updated: 2026-05-20
sources:
  - parcae-scaling-laws-looped-2026.md
  - loop-think-generalize-2026.md
status: draft
importance: medium
tags:
  - product
---

# OpenMythos

`kyegomez/OpenMythos` is an open-source reference implementation of [[recurrent-depth-transformer|recurrent-depth]] / [[looped-transformer|looped transformers]], built around the [[parcae|Parcae]] stabilization recipe (negative-diagonal `A` parameterization, prelude LayerNorm on input injection, per-sequence depth sampling) and the [[loop-think-generalize-2026|Loop, Think, Generalize]]-style training setup (zero-initialized loop boundaries, Poisson-sampled iteration counts, tail-only supervision for implicit reasoning). The repository has accumulated 13k+ GitHub stars and serves as the community baseline for experimenting with looped pretraining at small-to-mid scale. Notable derivatives include `25am093-crypto/Openmythos-2.0` (a continuation fork with updated training recipes) and `btrinos/nanochat-openmythos` (a [[nanochat]]-flavored integration that swaps in the looped block for Karpathy's reference architecture).

Link: <https://github.com/kyegomez/OpenMythos>

Cross-links: [[parcae]], [[recurrent-depth-transformer]], [[looped-transformer]].
