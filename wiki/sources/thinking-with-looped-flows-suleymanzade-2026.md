---
title: "Thinking with Looped Flows"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - thinking-with-looped-flows-suleymanzade-2026.md
arxiv_id: "2609.11801"
authors:
  - Ayhan Suleymanzade
  - Chanhyuk Lee
  - Floor Eijkelboom
  - Nicholas M. Boffi
  - İsmail İlkan Ceylan
  - Jinwoo Kim
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Thinking with Looped Flows

**Suleymanzade** (EPFL; visiting researcher at AITHYRA), **Lee** (KAIST), **Eijkelboom** (Amsterdam), **Boffi** ([[cmu|CMU]]), **Ceylan** (TU Wien, AITHYRA, Oxford), **Kim** (KAIST). [arXiv:2609.11801](https://arxiv.org/abs/2609.11801), cs.LG, v1 10 Sep 2026.

## Summary

Looped reasoners such as HRM, TRM and FPRM are trained with gradients cut between recurrence steps. That leaves early steps with no training signal for helping later ones, and it produces unstable recurrences. **Looped flows** make the denoiser of a categorical flow-matching model **stateful**: at each step it takes the noisy interpolant I_t = (1−t)x₀ + t·x₁ and the recurrent state z, predicts the clean solution, and updates z.

Training unrolls up to k steps. Each step has its own local cross-entropy loss, with stop-gradient between steps as in TRM. Two choices align the steps in time:

- The noise levels are **sorted so noise decreases** across the unroll.
- The **same noise and target pair is shared** across all steps.

An ACT head decides when to stop. Inference integrates the probability flow together with the recurrence, using an SDE sampler with noise backtracking. A finer time grid then becomes an extra axis for test-time compute, and different noise seeds give different valid solutions.

## Key points

- **Main results (Table 1, same TRM architecture: 5M MLP-Mixer for Sudoku, 7M transformer otherwise).**

  | Benchmark | Looped flows | TRM | Best other baseline |
  |---|---|---|---|
  | Sudoku-Extreme | **97.9 ± 0.4** | 87.4 | FPRM 94.2 (GRAM: no single-trajectory figure) |
  | Maze-Hard | 86.7 ± 1.1 | 85.3 | FPRM 87.0 (below this) |
  | ARC-AGI-1 | **58.8 ± 1.8** | 44.6 | GRAM 52.0 |
  | ARC-AGI-2 | **12.2 ± 1.9** | 7.8 | GRAM 11.1 |

  The baselines are published numbers. HRM scores 55.0 / 74.5 / 40.3 / 5.0 on the four benchmarks; FPRM scores 47.5 / 6.2 on ARC-1 / ARC-2.
- **Ensembling (5 trajectories, best-Q).** Sudoku 99.3, Maze 86.9, ARC-1 59.5, ARC-2 12.2. For comparison, PTRM reaches Sudoku 98.8 with 100 trajectories, EqR 99.8 with 128, and GRAM 97.0 with 20.
- **Inference-time scaling on Sudoku.** 74.5% at 8 steps rises to 97.9% at 128 steps, passing GRAM at 32 steps.
- **Stability.** Across about 65,000 Sudoku test instances, TRM fails on 12.6%. Of those failures, 88.3% never converge and 11.7% fall into spurious attractors. Looped flows fix 90.9% of TRM's failures: 89.9% of the non-converging cases and 98.0% of the attractor cases.
- **Multiple solutions (Table 2).** Looped flows are best on every accuracy, coverage and conflict metric for N-Queens and three-colouring, using 20 samples per instance.
- **Ablations (Tables 3–5).** Time conditioning, training on interpolants, decreasing noise, shared noise and stochastic integration each contribute. Self-conditioning is a special case of the method.

> [!note] Abstract vs body
> The abstract says "58.8% test accuracy on ARC-AGI-1". Table 1 reports **pass@2** for ARC, and "accuracy" only for Sudoku and Maze.

## What it does not show

- Only small models (5–7M parameters) on grid puzzles. There is no language or LLM setting.
- In principle the model could learn a shortcut from sharing noise across steps. Appendix B argues that it does not rely on one in practice; this is not proven.

## Concepts & entities

- [[looped-transformer]] · [[recurrence]] · [[recurrent-depth-transformer]] · [[latent-reasoning]] — the recurrent-reasoner family, and a training fix for it.
- [[adaptive-computation-time]] · [[test-time-compute]] — the ACT halting head, and time-grid refinement as a way to scale inference.
- [[diffusion-language-model]] · [[elf-embedded-language-flows-hu-2026]] · [[latent-reasoning-with-normalizing-flows-tu-2026]] — other flow-based reasoning work in the wiki.
- [[generative-recursive-reasoning-baek-2026]] — GRAM, the strongest stochastic looped baseline and the source of the multi-solution benchmarks.
- [[recurrent-looped-transformer-zhang-2026]] — a contrasting design that applies full BPTT through recurrence at LLM scale.
- [[cmu]].

## References

- [arXiv:2609.11801](https://arxiv.org/abs/2609.11801)
- HRM (Wang et al. 2025); TRM (Jolicoeur-Martineau 2025); FPRM (Movahedi et al. 2026); GRAM (Baek et al. 2026); PTRM (Sghaier et al. 2026); EqR (Huang et al. 2026). Unstable recurrences: Ren & Liu 2026.
- Categorical flow matching: Dieleman et al. 2022; Eijkelboom et al. 2024; Lee et al. 2026. SDE family: Song et al. 2021. ACT: Graves 2016.
