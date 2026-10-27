---
title: MDM (Human Motion Diffusion Model)
type: entity
created: 2026-10-24
updated: 2026-10-24
sources:
  - human-motion-diffusion-model-tevet-2022.md
tags:
  - model
status: draft
importance: medium
family: mdm
successors:
  - [[kimodo]]
  - [[unimate]]
release_date: 2022-09-29
---

# MDM (Human Motion Diffusion Model)

Tel Aviv University's 2022 text-to-motion diffusion model (ICLR 2023). Paper: [[human-motion-diffusion-model-tevet-2022]].
- A transformer encoder and a CLIP text encoder.
- **Predicts x₀**, which enables geometric losses.
- Classifier-free guidance (s = 2.5); editing by inpainting.
- 1,000 sampling steps, about a minute per sample.
- HumanML3D FID 0.544.

## Lineage

MDM is the design template that later systems inherit:
- [[kimodo]] keeps x₀-prediction, CFG and imputation, and scales data and parameters.
- [[unimate]] adds MDM-style text conditioning to topology-agnostic animation and switches to flow matching.
- The speed problem drove the move to fewer steps: 100 (Kimodo), 4–10 ([[ardy]]), 50 flow steps (UniMate).

## References

- [[human-motion-diffusion-model-tevet-2022]] · [[text-to-motion]]
