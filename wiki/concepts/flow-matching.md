---
title: Flow Matching
type: concept
created: 2026-10-24
updated: 2026-10-24
sources:
  - unimate-unified-skeleton-animation-mou-2026.md
status: draft
importance: medium
---

# Flow Matching

A generative-modelling objective (Lipman et al., 2022, [arXiv:2210.02747](https://arxiv.org/abs/2210.02747); not yet ingested). It trains a network to predict the **velocity** that carries noise to data along a chosen path. With a straight-line path (**rectified flow**), the target is simply `v* = x₁ − x₀`, and sampling integrates an ODE with a few Euler steps.

Compared with DDPM-style diffusion, there is no noise schedule to tune and sampling uses fewer, deterministic steps.

## In this wiki

- **[[unimate|UniMate]]** trains with rectified flow (velocity prediction, logit-normal time sampling). It adds a geodesic rotation loss computed on the one-step estimate `x̂₁ = x_τ + (1−τ)·v_θ`, and samples in 50 Euler steps ([[unimate-unified-skeleton-animation-mou-2026]]).
- **Contrast:** [[mdm|MDM]] and [[kimodo|Kimodo]] use DDPM/DDIM diffusion that predicts x₀ (1,000 and 100 steps).

## Related

[[text-to-motion]] · [[diffusion-language-model]]
