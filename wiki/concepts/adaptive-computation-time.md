---
title: Adaptive Computation Time (ACT)
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - adaptive-computation-time-graves-2016.md
status: complete
importance: high
tags:
  - 2016
---

# Adaptive Computation Time (ACT)

**Adaptive Computation Time** is the mechanism letting a recurrent or looped network decide *how many computation steps* to take per input element. Introduced by Alex Graves at [[deepmind|Google DeepMind]] in 2016 ([[adaptive-computation-time-graves-2016]]) for RNNs; later inherited by [[universal-transformer]] (Dehghani 2018) and a load-bearing tool for the entire [[looped-transformer]] / [[recurrent-depth-transformer]] thread.

## Core mechanism

- **Per-step halting probability head**: at each iteration `t`, the network emits a halt probability `h(t) ∈ [0, 1]`.
- **Halting rule**: stop once `Σ h(t) ≥ 1 − ε`, or after a max-step budget.
- **Ponder cost regularizer**: a penalty proportional to the number of steps taken — trades accuracy for compute, tuned by hyperparameter.
- **Differentiable through the halting decision** — gradients flow through both the network and the halt head.
- **Input-adaptive depth**: harder inputs take more steps; easier inputs halt early.

## What it advances

- First principled way to make compute scale with *input* difficulty rather than being a fixed architectural choice.
- The conceptual foundation for [[inference-time-compute|test-time-compute scaling]] in its modern (2024–2026) form — long before o1-style explicit-CoT scaling, ACT proposed scaling computation per-input.
- Reused in [[universal-transformer-dehghani-2018|Universal Transformer]] (per-position halting), then [[looped-transformers-length-generalization-fan-2024|Fan 2024]] (variable-step loops with confidence-based halting), and the "dynamic iteration" Poisson-sampling strategy in [[loop-think-generalize-2026]].

## Relation to other concepts

- [[looped-transformer]] / [[universal-transformer]] / [[recurrent-depth-transformer]] — all use ACT-style or ACT-inspired halting.
- [[overthinking]] (in [[loop-think-generalize-2026]]) — the failure mode that ACT-style halting is designed to mitigate.
- [[inference-time-compute]] — the modern umbrella concept ACT prefigures.

## References

- [[adaptive-computation-time-graves-2016]] — Graves 2016, the canonical paper
- [[universal-transformer-dehghani-2018]] — first transformer adoption
- [[looped-transformers-length-generalization-fan-2024]] — modern variant using confidence-based halting
