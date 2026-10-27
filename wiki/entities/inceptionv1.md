---
title: InceptionV1 (GoogLeNet)
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - feature-visualization-olah-2017.md
  - zoom-in-circuits-olah-2020.md
tags:
  - model
family: convnet
predecessor: [[alexnet]]
release_date: 2014
status: draft
importance: medium
---

# InceptionV1 (GoogLeNet)

InceptionV1, also called GoogLeNet (Szegedy et al., 2014), is an ImageNet [[convolutional-neural-network]] built from "inception" blocks. Each block runs parallel 1×1, 3×3 and 5×5 convolution branches, often with a 1×1 bottleneck in front. Its layers are named conv2d0…2 and mixed3a…mixed5b. Its own paper is not ingested.

It is the **reference organism of CNN interpretability**:
- **[[feature-visualization-olah-2017]]** uses it for optimization-based visualizations, including the fur-texture facets of a mixed4a neuron.
- **[[zoom-in-circuits-olah-2020]]** uses it for the [[circuits]] programme:
  - curve detectors in mixed3b;
  - high-low frequency detectors in early vision;
  - pose-invariant dog heads;
  - the polysemantic unit 4e:55 (cat faces, car fronts, cat legs);
  - the car feature in mixed4c that gets spread across dog neurons ([[superposition]]).

## Lineage

It succeeded [[alexnet|AlexNet]] (2012) on ImageNet and was followed by InceptionV3 and residual networks. Zoom In reports the same low-level features (curve and high-low frequency detectors) across AlexNet, InceptionV1, InceptionV3 and ResNets, which it offers as evidence for universality.

## References

- [[feature-visualization-olah-2017]] · [[zoom-in-circuits-olah-2020]] · [[feature-hierarchy]]
