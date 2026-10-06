---
title: "Scaling Laws, Carefully (Lil'Log)"
type: source
created: 2026-10-07
updated: 2026-10-07
sources:
  - scaling-laws-carefully-weng-2026.md
authors:
  - "[[lilian-weng]]"
year: 2026
venue: "Lil'Log"
url: https://lilianweng.github.io/posts/2026-06-24-scaling-laws/
tags:
  - 2026
  - blog
  - survey
status: complete
importance: high
---

# Scaling Laws, Carefully (Lil'Log)

**Source**: `sources/scaling-laws-carefully-weng-2026.md` ([lilianweng.github.io/posts/2026-06-24-scaling-laws](https://lilianweng.github.io/posts/2026-06-24-scaling-laws/))
**Author**: [[lilian-weng]], with feedback and edits credited to John Schulman
**Date**: **2026-06-24**

## Summary

A careful survey of [[scaling-laws|scaling laws]]: where they came from, why Kaplan and Chinchilla disagree, how the laws change when data runs out, and how fragile the fits are in practice. Notation: compute C ≈ 6ND, where 2ND is the forward pass and 4ND the backward pass.

## Contents

**Before "scaling laws" (learning-curve predictability):**
- Amari et al. (1992) derive four power-law learning curves.
- Hestness et al. (2017) find that architecture changes the *offset* of the power law, not the exponent. The exponent looks like a property of the problem domain. They also describe three phases: small-data, power-law, and irreducible-error.
- Rosenfeld et al. (2020) fit a joint error form L ≈ A/N^α + B/D^β + E on small models and extrapolate.

**Kaplan vs Chinchilla, reconciled:**
- Kaplan: N_opt ∝ C^0.73, meaning grow the model faster than the data. Chinchilla: N_opt ∝ C^0.5, meaning double the tokens with each doubling of parameters.
- Two reasons for the gap:
  1. Kaplan's experiments were on much smaller models, and small differences in fit grow large when extrapolated on log–log axes.
  2. Kaplan counted **non-embedding** parameters. Pearce & Song (2024) show that in Kaplan's 768M–1.5B range, the local exponent comes out near 0.73 even under the Chinchilla law, and converges to Chinchilla's at scale.

**Why a power law?**
- Sharma & Kaplan (2020): language modelling as regression on a low-dimensional data manifold.
- Michaud et al. (2023): **quantized skills** whose frequencies follow a power law. The model learns common skills first and rare ones later.

**The data-limited regime (the "data wall"):**
- Hernandez et al. (2022): repeated data causes double descent and hurts out-of-distribution and fine-tuning performance.
- **Muennighoff et al. (2023)**, about 400 runs: a repeated token's value **decays exponentially**, with a learned half-life r_D. Excess parameters lose value faster than repeated data does, so spend extra compute on more epochs rather than more parameters.
- **Lovelace et al. (2026)**, about 300 models: add an explicit **overfitting penalty** P·R_D^δ·(N/U_D)^κ. Larger models are *more* sensitive to repetition, and strong weight decay reduces the penalty.

**How fragile fits are in practice:**
- Predictions are sensitive to "seemingly trivial procedural choices": how parameters are counted, numerical precision, and how the loss is aggregated.
- The fits assume everything except scale stays fixed and well-tuned.
- A toy simulation shows how small fitting differences grow when extrapolated.

## Why it matters for the wiki

A trusted-source companion to [[scaling-laws]], [[scaling-laws-kaplan-2020]], [[chinchilla-hoffmann-2022]] and the [[kaplan-vs-chinchilla-vs-skaling]] comparison. Its reconciliation of the two laws (non-embedding parameter counts and small-model extrapolation) is the same kind of explanation that [[skaling-chinchilla-exponents-kaplan-coupling-videau-2026|Skaling]] formalizes. The data-constrained section is the wiki's first detailed treatment of the "data wall".

## Concepts & entities

[[scaling-laws]] · [[scaling-laws-kaplan-2020]] · [[chinchilla-hoffmann-2022]] · [[kaplan-vs-chinchilla-vs-skaling]] · [[pre-training]] · [[lilian-weng]]
