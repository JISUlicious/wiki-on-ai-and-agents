---
title: AlexNet
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - visualizing-understanding-convnets-zeiler-2013.md
  - transferable-features-deep-nets-yosinski-2014.md
tags:
  - model
family: convnet
successors:
  - [[inceptionv1]]
release_date: 2012
status: draft
importance: medium
---

# AlexNet

AlexNet (Krizhevsky, Sutskever & Hinton, 2012) is the 8-layer [[convolutional-neural-network]] that won ImageNet 2012 with **16.4% top-5 error, against 26.1%** for the runner-up. It was trained on GPUs with ReLUs and dropout, and it started the deep-learning era in vision ([[deep-learning]]). AlexNet's own paper is not ingested; these figures are as quoted in [[visualizing-understanding-convnets-zeiler-2013]].

## Layout

- 5 convolution layers: 11×11 stride-4 filters in the first, with max pooling and local contrast normalisation;
- 2 fully connected layers of 4,096 units each;
- a 1,000-way softmax.

## Lineage

**ZFNet** (Zeiler & Fergus 2013) is a modified AlexNet. Visualizing its layers showed two problems: aliasing from AlexNet's stride-4 first layer, and dead filters. The fix was 7×7 stride-2 first-layer filters, which gave 14.8% test top-5 as an ensemble. AlexNet's structure is also the reference network for the layer-transfer experiments in [[transferable-features-deep-nets-yosinski-2014]] (Caffe implementation). Its line continues to [[inceptionv1|InceptionV1/GoogLeNet]] (2014) and, beyond CNNs, to [[vit|ViT]].

## Why it matters here

AlexNet-style networks are the subject of the canonical [[feature-hierarchy]] evidence: edges in layer 1, textures in layer 3, object parts in layer 4, whole objects in layer 5. See [[layer-depth-hierarchy-cnn-vs-llm]].

## References

- [[visualizing-understanding-convnets-zeiler-2013]] · [[transferable-features-deep-nets-yosinski-2014]]
