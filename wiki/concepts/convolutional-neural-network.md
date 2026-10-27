---
title: Convolutional Neural Network
type: concept
created: 2026-10-27
updated: 2026-10-27
sources:
  - visualizing-understanding-convnets-zeiler-2013.md
  - transferable-features-deep-nets-yosinski-2014.md
  - zoom-in-circuits-olah-2020.md
status: draft
importance: medium
---

# Convolutional Neural Network

A convolutional neural network (CNN, convnet) stacks **convolution layers**. Each layer is a small learned filter slid over the spatial grid, followed by a ReLU and, optionally, pooling and normalisation. The model ends in fully connected layers and a softmax classifier.

The architecture goes back to LeCun et al. 1989. It became the default for vision after [[alexnet|AlexNet]] won ImageNet 2012 with 16.4% top-5 error against 26.1% for the runner-up ([[visualizing-understanding-convnets-zeiler-2013]]). It was later displaced in part by [[vit|Vision Transformers]] ([[vit-dosovitskiy-2020]]).

## Why CNNs form a spatial hierarchy

- **Local connectivity plus pooling makes the receptive field grow with depth.** A layer-1 unit sees a 7×7 patch. After repeated stride-2 convolutions and pooling, a layer-5 unit sees most of the image: in the ZFNet configuration the 224×224 input is reduced to a 6×6 grid by layer 5.
- **So the [[feature-hierarchy]] is partly forced by the architecture.** Early units *cannot* represent objects because they cannot see them.
- **What fills each level** ([[visualizing-understanding-convnets-zeiler-2013]], [[zoom-in-circuits-olah-2020]]):
  - edges and colour blobs (Gabor-like, essentially universal);
  - corners and curves;
  - textures;
  - object parts (eyes, wheels, dog faces);
  - whole objects with pose invariance.
- **Generality falls with depth** ([[transferable-features-deep-nets-yosinski-2014]]). Layers 1–2 transfer across tasks almost perfectly; the last layers are task-specific.

## Interpretability

CNNs were the first deep networks studied neuron by neuron:
- deconvnet projections (Zeiler & Fergus 2013);
- [[feature-visualization]] by optimization (Olah et al. 2017);
- the [[circuits]] programme on [[inceptionv1|InceptionV1]] (2020).

Many ideas in today's LLM [[mechanistic-interpretability]] began here: features as directions, [[polysemanticity]] and [[superposition]], and universality across models.

## Related

- [[layer-depth-hierarchy-cnn-vs-llm]]: how the CNN hierarchy compares with transformer LLM layers
- [[deep-learning]] · [[transfer-learning]] · [[vit]]

## References

- [[visualizing-understanding-convnets-zeiler-2013]] · [[transferable-features-deep-nets-yosinski-2014]] · [[zoom-in-circuits-olah-2020]] · [[feature-visualization-olah-2017]]
