---
title: "Cosmos 3: Omnimodal World Models for Physical AI"
type: source
created: 2026-06-19
updated: 2026-06-19
sources:
  - cosmos-3-omnimodal-world-models-for-physical-ai-nvidia-2026.md
arxiv_id: "2606.02800"
authors:
  - NVIDIA
  - et al.
first_author: NVIDIA
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Cosmos 3: Omnimodal World Models for Physical AI

Surfaced via the alphaXiv weekly digest (2026-06-10).

## Summary

Cosmos 3 is a family of omnimodal [[world-model]]s from [[nvidia]] that jointly process and generate language, image, video, audio, and action sequences within a single unified mixture-of-transformers architecture. The central argument is that Physical AI has historically fragmented into separate model classes — vision-language models for perception, video/forward-dynamics models for simulation, and vision-language-action (VLA) or world-action models for control — and that this separation is wasteful because understanding requires reasoning about future world evolution while generation requires compact structured world representations. By unifying perception, simulation, and execution without architectural modifications, Cosmos 3 subsumes all of these model classes as operational modes selected by input-output configuration.

The authors report state-of-the-art results across understanding and generation tasks. Post-trained checkpoints were ranked the best open-source Text-to-Image and Image-to-Video models by Artificial Analysis and the best policy model by RoboArena at the time of writing. Code, checkpoints, curated synthetic datasets (SDG-* for robots, driving, humans, warehouses), and the Cosmos-HUE evaluation benchmark are released openly under the Linux Foundation OpenMDW-1.1 license.

## Key points

- Single mixture-of-transformers backbone handles five modalities (language, image, video, audio, action) for both understanding and generation; flexible input-output config switches between modes (VLM, text-to-image, text/image/video-to-video, audio-video, world-action).
- Targets embodied / Physical AI: replaces a stitched pipeline (VLM plan + VLA action + forward-dynamics simulator) with one model that perceives, reasons, simulates, and acts.
- Cosmos3-Super and Cosmos3-Nano variants; Nano-Policy-DROID is a robot-policy checkpoint for real manipulation.
- Ranked best open-source Text-to-Image and Image-to-Video (Artificial Analysis) and best policy model (RoboArena) at report time.
- Ships large synthetic data-generation (SDG) datasets across physical-interaction, robot, driving, digital-human, and warehouse scenes, supporting data synthesis and policy learning.
- Fully open release: code, weights, datasets, and the Cosmos-HUE benchmark under OpenMDW-1.1.

## Concepts & entities

- [[nvidia]] — author lab
- [[world-model]] — the model family is framed as omnimodal world models
- Related: Physical AI, embodied AI, robotics, vision-language-action models (plain text — no dedicated pages)

## References

- [arXiv:2606.02800](https://arxiv.org/abs/2606.02800)
- Project: research.nvidia.com/labs/cosmos-lab/cosmos3; code at github.com/nvidia/cosmos
