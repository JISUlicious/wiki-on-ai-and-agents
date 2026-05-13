---
title: "An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale (ViT)"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - vit-dosovitskiy-2020.md
arxiv_id: "2010.11929"
venue: ICLR 2021
authors:
  - Alexey Dosovitskiy
  - Lucas Beyer
  - Alexander Kolesnikov
  - Dirk Weissenborn
  - Xiaohua Zhai
  - Thomas Unterthiner
  - Mostafa Dehghani
  - Matthias Minderer
  - Georg Heigold
  - Sylvain Gelly
  - Jakob Uszkoreit
  - Neil Houlsby
year: 2020
tags:
  - 2020
status: complete
importance: high
---

# An Image is Worth 16×16 Words: Transformers for Image Recognition at Scale

**Source**: `sources/vit-dosovitskiy-2020.md` (extracted from `ViT Dosovitsky 2020 2010.pdf`)
**arXiv**: [2010.11929](https://arxiv.org/abs/2010.11929)
**Venue**: ICLR 2021
**Authors**: Alexey Dosovitskiy, Lucas Beyer, Alexander Kolesnikov, ..., Jakob Uszkoreit (also of [[attention-is-all-you-need-vaswani-2017|the Transformer paper]]), Neil Houlsby ([[google-research|Google Research, Brain Team]])

## Summary

ViT (the Vision Transformer) shows that a **standard [[transformer-architecture|Transformer encoder]]**, applied directly to a sequence of image patches with **no convolution**, can match or exceed the best CNNs on image classification — provided it is pre-trained on sufficient data.

The recipe is striking in its minimalism. An image is split into fixed-size patches (default 16×16 pixels); each patch is flattened and linearly projected to produce a patch embedding; a learnable `[CLS]` token is prepended; learnable 1D positional embeddings are added; the resulting sequence is fed to a standard [[bert|BERT]]-style Transformer encoder. A classification head reads off the final `[CLS]` representation. That's it.

The paper's central empirical finding is that **inductive bias trades off with data scale**. CNNs have built-in translation equivariance and locality; Transformers do not. On mid-sized datasets like ImageNet alone, ViT slightly underperforms comparable ResNets. But on much larger pre-training corpora — public ImageNet-21k (14M images) or Google's in-house JFT-300M — ViT **overtakes** the best CNNs, reaching 88.55% on ImageNet, 94.55% on CIFAR-100, and 77.63% on VTAB. The lesson: at sufficient scale, the data teaches the model the locality and translation invariance that CNNs hard-code, and the more flexible Transformer comes out ahead.

ViT was the first compelling demonstration that the [[transformer-architecture|Transformer architecture]] is a *general* sequence model — not a language model — and is the foundation of the modern multimodal era (CLIP, DALL·E, Flamingo, GPT-4V).

## Key Points

- **Architecture**: standard Transformer encoder applied to image patches as tokens. No convolutions, no image-specific architecture priors.
- **Patchification**: H×W×C image → N = HW/P² patches of size P×P×C → linear projection to embedding dim. P = 16 in the default.
- **Inductive bias / scale trade-off**: ViT needs more data than CNNs to perform well, but with enough data it surpasses them.
- **Pre-training corpora**: JFT-300M (in-house, 300M images), ImageNet-21k (14M), or ImageNet-1k.
- **Variants**: ViT-Base (12L, 768d, 12h, 86M), ViT-Large (24L, 1024d, 16h, 307M), ViT-Huge (32L, 1280d, 16h, 632M).
- **Result**: 88.55% top-1 on ImageNet with ViT-Huge/14 pre-trained on JFT-300M.
- **Cheaper training than equivalent CNNs**: per the paper, ViT requires "substantially fewer computational resources" at equal performance.

## Entities Mentioned

- [[google-research]] — author affiliation (Brain Team)
- [[vit]] — the model

## Concepts Discussed

- [[transformer-architecture]] — applied unchanged to vision
- [[patch-tokenization]] — the new piece (treating images as token sequences)
- [[self-attention]] — over patches instead of words
- [[pre-training]] — at scale; central to the result
- [[scaling-laws]] — implicitly motivating the dataset scale-up

## Notable Quotes

> "We show that this reliance on CNNs is not necessary and a pure transformer applied directly to sequences of image patches can perform very well on image classification tasks." — Abstract

> "Large scale training trumps inductive bias." — §1

## References

_Original source: `sources/vit-dosovitskiy-2020.md`_
_Code: https://github.com/google-research/vision_transformer_
