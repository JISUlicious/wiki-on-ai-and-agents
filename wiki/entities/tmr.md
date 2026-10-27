---
title: TMR (Text-to-Motion Retrieval)
type: entity
created: 2026-10-24
updated: 2026-10-24
sources:
  - tmr-text-to-motion-retrieval-petrovich-2023.md
tags:
  - model
status: draft
importance: low
family: temos
---

# TMR (Text-to-Motion Retrieval)

A contrastive text–motion embedding (Petrovich, Black, Varol; ICCV 2023) built on TEMOS with InfoNCE and false-negative filtering. Paper: [[tmr-text-to-motion-retrieval-petrovich-2023]].

It now mainly serves as the **evaluator** for motion generation: R@k and FID are computed in its embedding space. [[kimodo]] ships `TMR-SOMA-RP-v1`, trained on 700 h of mocap, for its benchmark.

**Caveat:** an evaluator that can also generate scores its own model's outputs above real motion (92.44 vs 67.16 R@1).

## Lineage

TEMOS (VAE text-to-motion) → **TMR**, which adds contrastive retrieval → Kimodo's retrained TMR evaluator.

## References

- [[tmr-text-to-motion-retrieval-petrovich-2023]] · [[text-to-motion]]
