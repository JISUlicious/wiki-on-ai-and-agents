---
title: ARDY
type: entity
created: 2026-10-24
updated: 2026-10-24
sources:
  - ardy-autoregressive-motion-diffusion-zhao-2026.md
tags:
  - model
status: draft
importance: low
family: kimodo
predecessor: [[kimodo]]
parameters: ~156M (denoiser)
release_date: 2026-07-09
---

# ARDY

[[nvidia]] and ETH Zürich's **real-time autoregressive** motion diffusion model (SIGGRAPH 2026 / ACM TOG). Paper: [[ardy-autoregressive-motion-diffusion-zhao-2026]].
- **Speed:** 2-second windows at 20 fps, generated in **33 ms** (4 steps) or 63 ms (10 steps) on an RTX 4090.
- **Controls:** online text, root waypoints and paths, keyframes, end-effectors, and mouse/keyboard velocity.

## Lineage

[[kimodo]] (offline, 10 s clips) → **ARDY**:
- **Kept:** the two-stage root/body denoiser, the LLM2Vec text encoder, Rigplay data and imputation for constraints.
- **Changed:** an explicit root plus FSQ latent body tokens, autoregressive windows, and goals beyond the window.

Kimodo v1.2 ships an `ArdyRep` variant, so the two can share a benchmark.

## References

- [[ardy-autoregressive-motion-diffusion-zhao-2026]] · [[text-to-motion]]
