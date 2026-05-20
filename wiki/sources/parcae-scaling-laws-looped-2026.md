---
title: "Parcae: Scaling Laws for Stable Looped Language Models"
type: source
created: 2026-05-20
updated: 2026-05-20
sources:
  - parcae-scaling-laws-looped-2026.md
arxiv_id: "2604.12946"
authors:
  - Hayden Prairie
  - Zachary Novack
  - Taylor Berg-Kirkpatrick
  - Daniel Y. Fu
year: 2026
introduces:
  - "[[parcae]]"
  - "[[looped-language-model]]"
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Parcae: Scaling Laws for Stable Looped Language Models

[arXiv:2604.12946](https://arxiv.org/abs/2604.12946) — Prairie, Novack, Berg-Kirkpatrick, Fu (UCSD / Together AI), 2026. Companion blog: [sandyresearch.github.io/parcae](https://sandyresearch.github.io/parcae/).

## Summary

Parcae is the canonical 2026 scaling-laws paper for **looped language models** — transformer-style architectures that increase FLOPs by repeatedly passing activations through a shared block of layers, rather than by adding parameters. Prior looped recipes (RDMs [Geiping et al., 2025] and similar) had been observed to train *unstably* at scale: residual stream norms explode and the loss spikes, so practitioners relied on sensitive hyperparameter tuning and aggressive residual normalization. Parcae recasts the looped forward pass as a **nonlinear time-variant dynamical system over the residual stream** (`h_{t+1} = A h_t + B e + R(h_t, e)`); linearizing this yields a discrete LTI system whose stability is governed by the spectral norm of the injection matrix `A`. Empirically, divergent runs learn `ρ(A) ≥ 1`; convergent runs keep `ρ(A) < 1`.

The Parcae recipe stabilizes the loop with three load-bearing fixes: (1) a **negative-diagonal parameterization of A** discretized via ZOH / Euler with a learned step `Δ`, mechanically forcing `ρ(A) < 1` so the residual cannot explode; (2) a **prelude LayerNorm on the input injection `e`** to prevent the late-stage loss spikes that occur after ~170k steps; (3) **per-sequence depth sampling within a micro-batch** so the expectation over the variable-depth distribution `Λ` is faithfully estimated, plus parameterizing `Λ` by `µ_rec` alone (not `µ_bwd`) to preserve extrapolation to depths outside the training distribution. With these in place, residual normalization (Pre/Post-Norm hacks) is no longer needed.

Using the stabilized architecture, the paper establishes the first clean **scaling laws for looping** as an orthogonal axis to parameters and data. At fixed parameter count, FLOP-optimal training scales **looping and data in tandem** following empirical power laws (Chinchilla-style parabolic + parametric fits). At test time, additional loops yield a **saturating exponential decay** in loss toward an irreducible floor set by the training law, and the two laws unify into a single closed-form predictor accurate to ~1% on held-out 140M and 370M models. End-to-end, a 770M Parcae model matches a 1.3B Transformer on the [[core-benchmark|CORE]] benchmark; at 1.3B / 100B tokens, Parcae beats parameter-matched Transformers by 2.99 / 1.18 points on Core / Core-Extended and reduces validation perplexity by 4.3–9.2%. Against the prior RDM looped baseline, Parcae reduces validation PPL by up to 6.3% (and WikiText PPL by 9.1%).

This is the load-bearing reference for the [[openmythos]] open-source looped-LM stack, which reimplements the Parcae recipe.

## Key Points

- **Instability diagnosis.** Naive looped LMs (RDM-style with addition or concatenation injection) parameterize `A` such that `ρ(A) = 1` (marginally stable) or `ρ(A)` unconstrained (unstable). At scale this produces residual-state explosion (norms growing to ~10^17) and divergent training, which has historically been papered over with combined Pre+Post-Norm residual normalization plus careful LR sweeps.
- **LTI / dynamical-systems reframing.** Treating the looped forward pass as `h_{t+1} = A h_t + B e + R(h_t, e)` and linearizing (dropping `R`) reduces stability analysis to classical control theory: discrete LTI stability requires `ρ(A) < 1`. This is the analytical core of the paper.
- **Parcae's stabilization recipe (the load-bearing techniques):**
  1. Continuous parameterization `A := Diag(−exp(log A))` (negative diagonal), discretized via ZOH with a learned `Δ ∈ R^{d_h}`, so `A_disc = exp(Δ A)` automatically has spectral radius < 1.
  2. `B` left unconstrained but a **LayerNorm on the input embedding `e`** (a "prelude norm") to kill late-training loss spikes.
  3. **Per-sequence depth sampling** within a micro-batch (each sequence sees its own sampled loop count), better estimating the expectation over `Λ`.
  4. Parameterize `Λ` by `µ_rec` only (truncating by `µ_bwd` hurts extrapolation); set `µ_bwd = ⌈µ_rec / 2⌉`.
- **Hyperparameter robustness.** Across LRs 2e-4 → 1e-3, the baseline RDM converges only at 2e-4; Res-Norm RDMs at 2e-4 / 4e-4; **Parcae converges across all five LRs** (Table 2).
- **Training scaling law.** Parametric fit `L̂(N, D) = E + X·N^{−x} + Y·D^{−y}` extends naturally to looping `T` as a third axis. FLOP-optimal training scales **looping and data in tandem** at fixed parameter count.
- **Test-time scaling law.** Adding loops at inference produces a saturating exponential decay: `L(T) ≈ L_train(µ_rec, D) + Z · exp(−z · T / µ_rec)`. Unified train+test fit predicts within 0.85–1.31% average error on held-out models.
- **End-to-end quality vs. Transformers.** At 140M / 370M / 770M / 1.3B parameter scales (FineWeb-Edu, ~100B tokens, [[nanochat]]-matched arch), Parcae loops the middle third of layers and beats parameter-matched Transformers on Val PPL, Lambada PPL, CORE, and Core-Extended. The 770M Parcae ≈ 1.3B Transformer on CORE. Quality-gap-closure measured at 23.3–87.5% of the next-size Transformer.
- **End-to-end quality vs. RDM (prior looped).** Trained on Huginn dataset for apples-to-apples comparison: PPL reductions of up to 6.2% (val) / 9.1% (WikiText), Avg. zero-shot up ~1.8 points at T=16.
- **Practical implication.** Looping is now a usable, predictable third scaling axis alongside parameters and data — particularly attractive for inference-constrained / edge deployments where parameter budget (not FLOPs) is the binding constraint.
- **Limitations noted by authors.** Results limited to ≤1.3B / ~100B tokens; interplay among the three axes (params × data × recurrence) at frontier scale is open; test-time wall-clock cost grows with `µ_rec`.

## Entities

- **Authors:** Hayden Prairie, Zachary Novack, [[taylor-berg-kirkpatrick]], [[daniel-fu]] (Daniel Y. Fu)
- **Affiliations:** [[ucsd]] (University of California, San Diego); [[together-ai]] (Prairie and Fu are joint UCSD / Together AI)
- **Companion blog / open-source channel:** Sandy Research (`sandyresearch.github.io`) — Daniel Fu's group; basis for the [[openmythos]] open-source reimplementation.
- **Baselines:** RDM ([[recurrent-depth-model]], Geiping et al. 2025); [[nanochat]] ([[andrej-karpathy]]).

## Concepts

- [[looped-transformer]] / [[looped-language-model]] — the architectural family this paper stabilizes.
- [[parcae]] — the architecture introduced here.
- [[scaling-laws]] — extended to looping as a third axis alongside N and D.
- [[parameter-sharing]] — looping is the extreme case (one block reused T times).
- [[inference-time-compute]] / test-time scaling — looping as a knob for per-token adaptive compute.
- [[linear-time-invariant-system]] / dynamical systems view of residual streams — the analytical lens.
- [[layer-normalization]] — Parcae's prelude norm on `e`.
- [[chinchilla-scaling-laws]] — methodological template (parabolic + parametric Huber fits, L-BFGS).
- [[latent-reasoning]] — looped models as a substrate for reasoning in latent space (cited as motivation).
- Open-source implementation: [[openmythos]] — community reimplementation building on Parcae.

## Notable Quotes

> "Existing recipes for training looped architectures can be unstable, suffering from residual explosion and loss spikes. We address these challenges by recasting looping as a nonlinear time-variant dynamical system over the residual stream."

> "Via a linear approximation to this system, we find that instability occurs in existing looped architectures as a result of large spectral norms in their injection parameters."

> "Parcae … constrains the spectral norm of the injection parameters via discretization of a negative diagonal parameterization."

> "Our initial scaling laws suggest that looping and data should be increased in tandem, given a fixed FLOP budget."

> "When scaled up to 1.3B parameters, we find that Parcae improves CORE and Core-Extended quality by 2.99 and 1.18 points when compared to strong Transformer baselines under a fixed parameter and data budget, achieving a relative quality of up to 87.5% a Transformer twice the size."

> "Residual normalization is unnecessary when stability is properly controlled."

## References

- arXiv: <https://arxiv.org/abs/2604.12946>
- Companion blog (Sandy Research): <https://sandyresearch.github.io/parcae/>
- Baseline RDM (looped LM): Geiping et al., 2025 ([[recurrent-depth-model]])
- Transformer baseline harness: Karpathy, [nanochat](https://github.com/karpathy/nanochat), 2025
- Open-source reimplementation: [[openmythos]]
