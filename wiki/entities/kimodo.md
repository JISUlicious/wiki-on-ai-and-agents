---
title: Kimodo
type: entity
created: 2026-10-24
updated: 2026-10-24
sources:
  - kimodo-kinematic-motion-diffusion-nvidia-2026.md
tags:
  - model
status: draft
importance: medium
family: kimodo
builds-on:
  - [[mdm]]
successors:
  - [[ardy]]
parameters: 282M (denoiser) + LLM2Vec-Llama-3-8B text encoder
release_date: 2026-03-16
---

# Kimodo

[[nvidia]]'s **ki**nematic **mo**tion **d**iffusi**o**n model. It is trained on 700 h of optical mocap and generates human (SOMA 77-joint, SMPL-X) and humanoid-robot (Unitree G1) motion from text plus kinematic constraints. Paper: [[kimodo-kinematic-motion-diffusion-nvidia-2026]].

**Repo:** [nv-tlabs/kimodo](https://github.com/nv-tlabs/kimodo), Apache-2.0 code.
- **Tools:** `kimodo_gen` (CLI) and `kimodo_demo` (a web timeline editor on 127.0.0.1:7860).
- **Outputs:** MuJoCo CSV and AMASS formats.
- **Benchmark:** 22,474 test cases.

**Model versions:**

| Version | Date | Notes |
|---|---|---|
| v1 | 2026-03-16 | five variants: SOMA, G1, SMPL-X |
| v1.1 | 2026-04-10 | benchmark release |
| **v1.2** | 2026-10-15 | **retrained after BONES-SEED test sequences were found in v1.1's training set**; adds an ARDY-representation variant |

**Hardware:** about 17 GB VRAM because of the text encoder, under 3 GB with `TEXT_ENCODER_DEVICE=cpu`.

## Lineage

- **Inherits from [[mdm|MDM]]:** the transformer denoiser that predicts x₀, classifier-free guidance, and imputation for constraints.
- **Changes:**
  - a two-stage root/body denoiser;
  - LLM2Vec in place of CLIP;
  - separated text and constraint guidance;
  - 100-step DDIM.
- **Scales:** data about 20× (700 h) and parameters to 282M.
- **Successor:** [[ardy]] makes it real-time and streaming.
- **Downstream:** [[sonic|GEAR-SONIC]] tracks its G1 output in a demo.

## References

- [[kimodo-kinematic-motion-diffusion-nvidia-2026]] · [[text-to-motion]]
