---
title: Text-to-Motion Generation
type: concept
created: 2026-10-24
updated: 2026-10-24
sources:
  - human-motion-diffusion-model-tevet-2022.md
  - tmr-text-to-motion-retrieval-petrovich-2023.md
  - kimodo-kinematic-motion-diffusion-nvidia-2026.md
  - ardy-autoregressive-motion-diffusion-zhao-2026.md
  - unimate-unified-skeleton-animation-mou-2026.md
status: draft
importance: medium
---

# Text-to-Motion Generation

Generating **3D skeletal motion** (joint rotations, positions and root trajectory over time) from a text prompt, often with extra **kinematic constraints**: keyframes, root paths, hand and foot targets. It serves animation and games, and increasingly **humanoid robotics**, where a kinematic motion is handed to a physics tracking policy ([[humanoid-motion-tracking]]).

## The lineage in this wiki

| Model | Year | Generator | Data | Control | Speed |
|---|---|---|---|---|---|
| [[mdm\|MDM]] | 2022 | diffusion, predicts x₀, CLIP text | HumanML3D (about 30 h) | text; inpainting edits | 1,000 steps, about 1 min |
| [[kimodo\|Kimodo]] | 2026 | two-stage diffusion, LLM2Vec, 282M | 700 h studio mocap | text + full-body, end-effector, root and contact constraints | 100 DDIM steps, 2–5 s per clip (offline) |
| [[ardy\|ARDY]] | 2026 | autoregressive diffusion, hybrid explicit/latent tokens | same as Kimodo | online text, constraints, mouse/keyboard | 4–10 steps, 33–63 ms per window (real time) |
| [[unimate\|UniMate]] | 2026 | [[flow-matching\|flow matching]], topology-aware DiT, FLAN-T5 | UniML3D, about 13K clips on thousands of skeletons | text only, **any skeleton** | 50 Euler steps, about 50 FPS |

## Recurring design choices

- **Predict the clean motion (x₀) or the velocity, not the noise**, so geometric terms such as forward kinematics, foot contact and rotation geodesics can be added to the loss.
- **Classifier-free guidance.** Kimodo **separates** text guidance from constraint guidance.
- **Imputation for control:** overwrite known values in the noisy input and add a mask. This needs no ControlNet and no test-time optimization.
- **The text encoder got bigger:** CLIP (MDM) → FLAN-T5 (UniMate) → LLM2Vec on Llama-3-8B (Kimodo, ARDY). Kimodo's ~17 GB VRAM is mostly the text encoder.
- **Foot skate is the persistent artifact.** Most systems add IK foot-locking post-processing.

## Evaluation and its pitfalls

- **Standard metrics:** FID, R-precision and multimodal distance in a text–motion embedding. [[tmr|TMR]] is the modern evaluator.
- **An evaluator that can generate favours its own model** (TMR's own generator scores 92.44 vs 67.16 for real motion).
- **Benchmarks drift:**
  - Kimodo found BONES-SEED test sequences in a released model's training set;
  - it had a metric-averaging bug;
  - TF32 changed its foot-skate numbers.

  All three were corrected in its changelog ([[kimodo-kinematic-motion-diffusion-nvidia-2026]]). HumanML3D is considered saturated.

## Related

[[humanoid-motion-tracking]] · [[flow-matching]] · [[vision-language-action-model]] · [[diffusion-language-model]] (the same denoising machinery applied to text)
