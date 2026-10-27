---
title: UniMate
type: entity
created: 2026-10-24
updated: 2026-10-24
sources:
  - unimate-unified-skeleton-animation-mou-2026.md
tags:
  - model
status: draft
importance: medium
family: unimate
builds-on:
  - [[mdm]]
  - [[flow-matching]]
release_date: 2026-09-06
---

# UniMate

A text-to-animation model for **arbitrary skeletons**: bipeds, quadrupeds, birds, fish, snakes and articulated objects. It comes from Princeton, Berkeley, MIT and NTU (SIGGRAPH Asia 2026). Paper: [[unimate-unified-skeleton-animation-mou-2026]].

**Repo:** [Friedrich-M/UniMate](https://github.com/Friedrich-M/UniMate). Code is MIT; checkpoints (`Linzhan/UniMate`) are CC BY-NC 4.0; the dataset is UniML3D. Training and inference code was released 2026-09-06, and preview checkpoints 2026-09-27. Since 2026-10-05, `rig_preprocess` turns any rigged GLB, glTF or FBX into an input.

**How it works:**
- A topology-aware DiT with factorized joint/time attention.
- Topology enters through graph-distance biases, Spec-RoPE (Laplacian eigenvectors) and joint-name embeddings.
- A flow-matching objective, a frozen FLAN-T5-Base text encoder, and 50-step sampling at about 50 FPS.

## Lineage

- **From [[mdm|MDM]]:** the text-conditioned motion transformer and classifier-free guidance.
- **From AnyTop:** topology-agnostic motion; the repo partly builds on AnyTop's code.
- **What changed:** a switch from x₀-prediction diffusion to **[[flow-matching|flow matching]]**, and one model across thousands of skeletons rather than one per skeleton.
- **Contrast:** [[kimodo]] goes the other way, scaling *human-only* data with rich kinematic constraints.

## References

- [[unimate-unified-skeleton-animation-mou-2026]] · [[text-to-motion]]
