---
title: ViT (Vision Transformer)
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - vit-dosovitskiy-2020.md
family: transformer
predecessor: "[[transformer]]"
successors: []
parameters: "86M (Base) / 307M (Large) / 632M (Huge)"
release_date: 2020-10-22
status: complete
importance: high
tags:
  - model
---

# ViT (Vision Transformer)

The Vision Transformer (ViT) is the image-classification model introduced in "[[vit-dosovitskiy-2020|An Image is Worth 16×16 Words]]" (Dosovitskiy et al., ICLR 2021). It applies a standard [[transformer-architecture|Transformer encoder]] — essentially [[bert|BERT]]'s architecture — directly to a sequence of image patches.

## Architecture

1. Split an image of shape `(H, W, C)` into N patches of size `(P, P, C)` (default `P=16` → 14×14=196 patches for a 224×224 image).
2. Flatten each patch to a vector of length `P²·C` and linearly project to embedding dim `D`.
3. Prepend a learnable `[CLS]` token.
4. Add learnable 1D positional embeddings.
5. Feed through a standard Transformer encoder (`L` layers of [[multi-head-attention|MHSA]] + MLP, with [[layer-normalization|LayerNorm]] and [[residual-connection|residuals]]).
6. Read off the final `[CLS]` representation, apply a classification MLP head.

| Variant | Layers (L) | dmodel (D) | Heads | Params |
|---|---|---|---|---|
| ViT-Base/16 | 12 | 768 | 12 | 86M |
| ViT-Large/16 | 24 | 1024 | 16 | 307M |
| ViT-Huge/14 | 32 | 1280 | 16 | 632M |

(The `/N` denotes patch size N.)

## Training

- **Pre-training**: JFT-300M (300M Google-internal images) or ImageNet-21k (14M).
- **Fine-tuning**: on each downstream classification dataset.
- Supervised classification objective throughout (this is *not* self-supervised; subsequent work like MAE adopted MLM-style training).

## Lineage

| Predecessor | Successor |
|---|---|
| [[transformer]] (Vaswani 2017) | Downstream multimodal models (CLIP, ALIGN, Flamingo, GPT-4V) all use a ViT-style image encoder. Self-supervised variants (DINO, MAE, BEiT). |

ViT was the first definitive proof that the [[transformer-architecture|Transformer]] is a *general* sequence model — not a language model — and is the architectural ancestor of every modern vision-language model.

## Results (at release)

- ImageNet top-1: **88.55%** (ViT-Huge/14, pre-trained on JFT-300M).
- ImageNet-ReaL: 90.72%.
- CIFAR-100: 94.55%.
- VTAB (19 tasks): 77.63%.

## References

- [[vit-dosovitskiy-2020]]
