---
title: "Human Motion Diffusion Model (MDM)"
type: source
created: 2026-10-24
updated: 2026-10-24
sources:
  - human-motion-diffusion-model-tevet-2022.md
tags:
  - 2022
  - paper
status: complete
importance: high
authors:
  - Guy Tevet
  - Sigal Raab
  - Brian Gordon
  - Yonatan Shafir
  - Daniel Cohen-Or
  - Amit H. Bermano
introduces:
  - [[mdm]]
  - [[text-to-motion]]
year: 2022
venue: ICLR 2023
arxiv_id: "2209.14916"
---

# Human Motion Diffusion Model (MDM)

[arXiv:2209.14916](https://arxiv.org/abs/2209.14916), Tel Aviv University, 2022. ICLR 2023 is the venue as cited by later papers; it isn't stated in the PDF.

## What it established

[[mdm|MDM]] was the first widely adopted **diffusion** model for [[text-to-motion]], replacing VAE/autoencoder approaches (TEMOS, T2M). Several of its choices became the field's template:
- **A transformer encoder** over frames, with timestep and condition summed into one prepended token. It has 8 layers and width 512, and the paper finds it "not very sensitive to the choice of architecture".
- **A frozen CLIP ViT-B/32 text encoder.**
- **Predict the clean motion x̂₀, not the noise ε.** That makes geometric losses on positions, foot contact and velocity possible. On HumanML3D they are skipped, because positions and contacts are already explicit in the representation.
- **Classifier-free guidance:** the condition is dropped for 10% of training samples, and s = 2.5 is the sweet spot.
- **Editing by inpainting:** in-betweening, and body-part editing such as regenerating only the upper body.
- **Sampling:** 1,000 diffusion steps with a cosine schedule. Training takes about 3 days on one RTX 2080 Ti.

## Results

| HumanML3D | FID ↓ | R-precision top-3 ↑ | Diversity (real 9.503) |
|---|---|---|---|
| **MDM** | **0.544** | 0.611 | 9.559 |
| T2M | 1.067 | **0.740** | — |

- **MDM wins on realism, loses on text matching.** It has better FID and diversity but worse R-precision. KIT shows the same pattern: FID 0.497 vs 2.770, R-precision 0.396 vs 0.693.
- **User study** (KIT, 31 users): MDM was preferred over T2M 54.8% of the time, and even over ground truth 42.3% of the time.

## Limitation

Slow sampling: about 1,000 forward passes, "about a minute" per sample. Later systems cut this to 100 DDIM steps ([[kimodo]]), 4–10 steps autoregressively ([[ardy]]) and 50 flow steps ([[unimate]]).

## Related

[[mdm]] · [[text-to-motion]] · [[tmr-text-to-motion-retrieval-petrovich-2023]] · [[unimate-unified-skeleton-animation-mou-2026]] · [[kimodo-kinematic-motion-diffusion-nvidia-2026]]
