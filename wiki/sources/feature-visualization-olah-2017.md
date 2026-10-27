---
title: "Feature Visualization (Olah, Mordvintsev & Schubert, Distill)"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - feature-visualization-olah-2017.md
tags:
  - 2017
  - distill
status: complete
importance: medium
authors:
  - [[chris-olah]]
  - Alexander Mordvintsev
  - Ludwig Schubert
introduces:
  - [[feature-visualization]]
year: 2017
venue: "Distill (doi:10.23915/distill.00007), 2017-11-07"
---

# Feature Visualization (Olah, Mordvintsev & Schubert, Distill)

**Source**: `sources/feature-visualization-olah-2017.md`, with the HTML tags stripped from [distill.pub/2017/feature-visualization](https://distill.pub/2017/feature-visualization/).
- **Authors:** [[chris-olah|Chris Olah]] (Google Brain), Alexander Mordvintsev (Google Research) and Ludwig Schubert (Google Brain).
- **Published:** 2017-11-07.
- **Subject model:** GoogLeNet ([[inceptionv1|InceptionV1]]) trained on ImageNet.

## Summary

This is a practical guide to **[[feature-visualization]] by optimization**. Start from noise and use gradients to change the *input image* until a chosen unit fires strongly. It is one of the two main threads of interpretability; the other is attribution, which asks which part of an input caused an activation.

- **What to optimise.** The target can be a single neuron (one position in a channel), a whole channel (the default in the article), a whole layer (the DeepDream objective), or a class logit or probability. Optimising pre-softmax logits gives better images, because raising a probability is easiest by suppressing the other classes.
- **Why optimise instead of using dataset examples.** Optimization "separates the things causing behavior from things that merely correlate with the causes". Dataset examples remain essential for seeing the whole range of what a neuron responds to.
- **Diversity.** A neuron can have several *facets*. Adding a diversity term (negative pairwise cosine similarity of Gram matrices) shows them.
  - Example: a neuron in layer mixed4a that a single visualization makes look like "top of dog heads" turns out, with diversity, to be mostly about **fur texture**.
  - In higher layers, diversity shows *different types of objects* that excite one neuron.
- **Neurons vs directions.** Do individual neurons (the basis directions) mean more than random directions in activation space?
  - Szegedy et al. found random directions equally meaningful; Bau, Zhou et al. found the basis directions interpretable more often.
  - The authors' experience: "random directions often seem interpretable, but at a lower rate than basis directions".
  - This anticipates the random-direction controls in [[neuron-explorer-lyramakesmusic-2026]].
- **The main obstacle is high-frequency noise.** Unregularised optimization produces adversarial-looking patterns, partly because strided convolutions and pooling leave checkerboard patterns in the gradient. Three families of fixes:
  - frequency penalisation (total variation, blurring);
  - transformation robustness (jitter, rotate, scale);
  - learned priors (generative models).

  Preconditioning the optimization in a decorrelated Fourier basis helps a lot.
- **Open problems named:** how neurons interact, which units are most meaningful, and a holistic view of a feature's facets.

> [!note] Layer progression not captured in the text
> The article's figures show visualizations across GoogLeNet layers (the text names mixed4a, mixed4e and mixed5a), but the per-layer captions are images or scripts that the text extraction did not capture. The wiki's sourced statements of the edges → textures → parts → objects progression come from [[visualizing-understanding-convnets-zeiler-2013]] and [[zoom-in-circuits-olah-2020]].

## References

- [[feature-visualization]] · [[inceptionv1]] · [[convolutional-neural-network]] · [[polysemanticity]] · [[mechanistic-interpretability]]
- [[zoom-in-circuits-olah-2020]] · [[layer-depth-hierarchy-cnn-vs-llm]]
