---
title: DINOv2
type: entity
created: 2026-11-03
updated: 2026-11-11
sources:
  - platonic-representation-hypothesis-jepas-balestriero-2026.md
  - e-means-outperforming-k-means-bessa-2026.md
family: dino
builds-on:
  - [[vit]]
parameters: 21M (ViT-S) / 86M (ViT-B) / 300M (ViT-L) / 1.1B (ViT-g)
release_date: 2023-04-14
status: draft
importance: medium
tags:
  - model
---

# DINOv2

A family of self-supervised [[vit|Vision Transformer]] image encoders from [[meta|Meta AI]] (Oquab et al., "DINOv2: Learning Robust Visual Features without Supervision", [arXiv:2304.07193](https://arxiv.org/abs/2304.07193), April 2023). It is trained without labels or captions, by self-distillation between different crops of the same image, on a curated set of about 142M images. Its frozen features are widely used as a general-purpose vision backbone.

## Lineage

- **Predecessor:** DINO (Caron et al., 2021), the original self-distillation-with-no-labels method on ViTs.
- **What changed:** a much larger, automatically curated pretraining set and a combination of image-level and patch-level objectives, scaled up to a 1.1B-parameter ViT-g.
- **Successor:** DINOv3 (2025), which the 2026 PRH paper also uses in its captioner. It has no page here yet.

## In this wiki

- **The reference vision model for the [[platonic-representation-hypothesis]].** Huh et al. (2024) measured language–vision alignment against DINOv2, and [[platonic-representation-hypothesis-jepas-balestriero-2026]] keeps it for comparability.
- In that paper's terms DINOv2 is a "bad-view" [[jepa|JEPA]]-like learner, because its positives are crops of one photo. Its 109M good-view text JEPA reaches .184 mutual 10-NN with DINOv2 ViT-B on WIT-1024, above every language model the authors evaluated.

The model details on this page come from the DINOv2 paper, which has no source page in this wiki.

## References

- [[platonic-representation-hypothesis-jepas-balestriero-2026]]
- [DINOv2, arXiv:2304.07193](https://arxiv.org/abs/2304.07193)

## Newer evidence (2026-11-11)

- DINO10B, a 10-billion-vector dataset of DINO features (1024 dimensions), was released with e-means as a benchmark for clustering and nearest-neighbour search at scale. ([[e-means-outperforming-k-means-bessa-2026]])
