---
title: "Visualizing and Understanding Convolutional Networks (Zeiler & Fergus)"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - visualizing-understanding-convnets-zeiler-2013.md
tags:
  - 2013
  - paper
status: complete
importance: high
authors:
  - Matthew D. Zeiler
  - Rob Fergus
introduces:
  - [[feature-hierarchy]]
year: 2013
venue: "arXiv preprint (cs.CV); published at ECCV 2014 (venue from external record)"
arxiv_id: "1311.2901"
---

# Visualizing and Understanding Convolutional Networks (Zeiler & Fergus)

**Source**: `sources/visualizing-understanding-convnets-zeiler-2013.md`, text extracted from [arXiv:1311.2901](https://arxiv.org/abs/1311.2901) (v3, 2013-11-28). Authors: Matthew D. Zeiler and Rob Fergus, both at NYU (Courant Institute).

## Summary

This paper made the **layer-by-layer feature hierarchy** of [[convolutional-neural-network|convnets]] visible. It attaches a **deconvnet** to each layer of an [[alexnet|AlexNet]]-style ImageNet model. The deconvnet runs the network backwards to the image: it undoes max pooling using recorded "switch" locations, applies the ReLU, and filters with transposed weights. This projects any single feature activation back to the pixels that caused it.

The projections show features that are "far from random, uninterpretable patterns". In the authors' words, they show "compositionality, increasing invariance and class discrimination as we ascend the layers".

## What each layer responds to (Fig. 2; ImageNet validation set, top 9 activations per feature map)

| Layer | Responds to |
|---|---|
| 1 | Simple filters: oriented edges and colours (Fig. 6) |
| 2 | Corners and other edge/colour conjunctions |
| 3 | More complex invariances: similar **textures** (mesh patterns, text) |
| 4 | Significant variation, more class-specific: **dog faces, bird legs** |
| 5 | **Entire objects** with significant pose variation (keyboards, dogs) |

**Context matters too.** One layer-5 feature map turns out to respond to the *grass in the background*, not to the foreground objects.

## Other layer-wise findings

- **Training dynamics.** Lower layers converge within a few epochs. Upper layers only develop after 40–50 epochs.
- **Invariance grows with depth** (Fig. 5). Small translations and scalings change layer-1 features dramatically but layer-7 features only a little, and roughly linearly. The output is not invariant to rotation, except for rotationally symmetric objects.
- **Features get more discriminative with depth** (Table 7). The authors froze the first *n* layers and trained a linear SVM on them:

  | Layers used | 1 | 2 | 3 | 4 | 5 | 7 |
  |---|---|---|---|---|---|---|
  | Caltech-101 (30 per class) | 44.8 | 66.2 | 72.3 | 76.6 | 86.2 | 85.5 |
  | Caltech-256 (60 per class) | 24.6 | 39.6 | 46.0 | 51.3 | 65.6 | 71.7 |

  "a steady improvement can be seen as we ascend the model". On Caltech-101 the gain flattens after layer 5.
- **Depth matters more than any single layer** (Table 3). Removing the fully connected layers 6–7, or the middle conv layers 3–4, raises error only slightly. Removing both, which leaves a 4-layer model, is "dramatically worse".
- **The model finds the object.** Occluding the object makes the correct-class probability fall sharply, and its strongest layer-5 feature map goes quiet. So the model localises the object instead of relying on scene context.
- **Correspondence between parts.** Masking the same facial part (left eye, right eye, nose) across five dog images changes layer-5 features more consistently than masking random regions. This is implicit part correspondence. At layer 7 the effect is weaker.

## Visualisation as debugging

The visualisations exposed problems with AlexNet:
- its layer-1 filters mixed extreme high and low frequencies, leaving the middle frequencies uncovered;
- its layer-2 features showed aliasing from the stride of 4.

The fix was a 7×7 first-layer filter with stride 2. That gave cleaner features and beat AlexNet's single-model result by 1.7% (test top-5). An ensemble reached **14.8% test top-5**, the best published result at the time.

Transfer was strong: training a new softmax on frozen ImageNet features gave **86.5% on Caltech-101** and **74.2% on Caltech-256**, against 55.2% for the previous best at 60 images per class. That made it an early demonstration of [[transfer-learning]] from ImageNet.

## Relevance to this wiki

This is the canonical source for "early layers see edges, later layers see objects". [[layer-depth-hierarchy-cnn-vs-llm]] compares this picture with what LLM layers do.
- **What drives it:** the hierarchy comes from the growing receptive field of stacked convolutions and pooling. This paper's model goes from 224×224 down to 6×6×256 before the fully connected layers.
- **The tool:** deconvnet projections are an ancestor of [[feature-visualization]].

## References

- [[convolutional-neural-network]] · [[feature-hierarchy]] · [[alexnet]] · [[transfer-learning]] · [[feature-visualization]]
- [[transferable-features-deep-nets-yosinski-2014]]: the transfer-by-layer follow-up
- [[layer-depth-hierarchy-cnn-vs-llm]]
