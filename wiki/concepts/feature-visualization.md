---
title: Feature Visualization
type: concept
created: 2026-10-27
updated: 2026-10-27
sources:
  - feature-visualization-olah-2017.md
  - visualizing-understanding-convnets-zeiler-2013.md
  - zoom-in-circuits-olah-2020.md
status: draft
importance: medium
---

# Feature Visualization

Feature visualization explains a unit (a neuron, channel, layer or class) by **producing an input that makes it fire**. It is one of the two main threads of interpretability; the other is attribution, which asks which part of an input caused an activation ([[feature-visualization-olah-2017]]).

## Methods

- **Dataset examples.** Find the real inputs with the highest activation. They show the range of what a unit responds to, but mix true causes with things that merely correlate with them.
- **Deconvnet projection** ([[visualizing-understanding-convnets-zeiler-2013]]). Map one activation back to the pixels of the image that caused it.
- **Optimization.** Start from noise and use gradients to change the input until the target activates. This isolates causes.
  - It needs regularisation (frequency penalties, robustness to transformations, learned priors, Fourier-basis preconditioning) to avoid adversarial high-frequency noise.
  - It needs a diversity term to show a unit's different facets.

## In language models

Input images can't be optimised in token space the same way, so LLM work uses other tools:
- top-activating text examples;
- [[logit-lens]] readouts of a neuron's output weights;
- [[sparse-autoencoder|SAE]] features;
- **steering**: push the unit and read what the model writes ([[neuron-explorer-lyramakesmusic-2026]]). This is the LLM analogue of "what input makes it fire", turned around to "what output does it cause".

## References

- [[feature-visualization-olah-2017]] · [[visualizing-understanding-convnets-zeiler-2013]] · [[zoom-in-circuits-olah-2020]]
- [[feature-hierarchy]] · [[circuits]] · [[mechanistic-interpretability]]
