---
title: "Zoom In: An Introduction to Circuits (Olah et al., Distill)"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - zoom-in-circuits-olah-2020.md
tags:
  - 2020
  - distill
status: complete
importance: high
authors:
  - [[chris-olah]]
  - [[nick-cammarata]]
  - Ludwig Schubert
  - Gabriel Goh
  - Michael Petrov
  - Shan Carter
introduces:
  - [[circuits]]
year: 2020
venue: "Distill, Circuits thread (doi:10.23915/distill.00024.001), 2020-03-10"
---

# Zoom In: An Introduction to Circuits (Olah et al., Distill)

**Source**: `sources/zoom-in-circuits-olah-2020.md`, with the HTML tags stripped from [distill.pub/2020/circuits/zoom-in](https://distill.pub/2020/circuits/zoom-in/).
- **Authors:** [[chris-olah|Chris Olah]], [[nick-cammarata|Nick Cammarata]], Ludwig Schubert, Gabriel Goh, Michael Petrov and Shan Carter, all at [[openai|OpenAI]] at the time.
- **Published:** 2020-03-10, as the opening essay of the Circuits thread.
- **Subject model:** [[inceptionv1|InceptionV1]].

## Summary

The essay proposes "zooming in": studying single neurons and even single weights with the care of cellular biology. Its framework is three deliberately speculative claims, modelled on Schwann's cell theory:

1. **Features.** "Features are the fundamental unit of neural networks. They correspond to directions" (a linear combination of neurons). "Early layers contain features like edge or curve detectors, while later layers have features like floppy ear detectors or wheel detectors."
2. **[[circuits|Circuits]].** Features connected by weights form subgraphs that "can also be rigorously studied and understood". You can "literally read meaningful algorithms off of the weights."
3. **Universality.** Analogous features and circuits form across models and tasks. The evidence offered is admitted to be "anecdotal".

## Features, by depth

- **Curve detectors (mixed3b, an early layer).**
  - They respond to curves with a radius of about 60 px, in **families** that cover every orientation.
  - Seven lines of evidence back the label: feature visualization, dataset examples, synthetic stimuli, joint tuning, how the feature is built, how it is used, and a **hand-written reimplementation** that mimics the original.
  - They "straddle the boundary" between features everyone accepts (edges) and contested high-level ones.
- **High-low frequency detectors (early vision).** They look for low frequency on one side of the receptive field and high frequency on the other. This is a boundary heuristic for out-of-focus backgrounds that no one predicted in advance.
- **A pose-invariant dog-head detector** in a higher layer.
- **[[polysemanticity|Polysemantic neurons]].** Unit **4e:55** responds to cat faces, car fronts and cat legs. It is not tracking a shared feature; it detects each separately. "If one neuron with five different meanings connects to another neuron with five different meanings, that's effectively 25 connections."

## Circuits, by depth

- **Curves, spanning 2 layers.** Full curve detectors are built from earlier curve and line detectors arranged as "tangent curves". Opposite orientations inhibit. The weights rotate with the orientation, a motif the authors call **equivariance**. Downstream, the curves feed circles, spirals and 3D-curvature detectors.
- **Oriented dog heads, spanning 4 layers.** Two mirrored pathways (facing left, facing right) inhibit each other and are then merged into invariant units. The authors call this **"unioning over cases"**; it has some XOR-like properties.
- **Cars in superposition (mixed4c, a mid-to-late layer).** A clean car neuron, built from windows above and wheels below, is then *spread across* dog-detecting neurons in the next layer. This is **[[superposition]]**: it saves neurons because "high-dimensional spaces … only allow for n orthogonal vectors, but exponentially many almost orthogonal vectors". It suggests polysemantic neurons are in some sense deliberate.

## Universality

- The same low-level features (curve detectors, high-low frequency detectors) appear in AlexNet, InceptionV1, InceptionV3, ResNets, and in models trained on Places365.
- Gabor filters in layer 1 are "widely accepted".

## Relevance to this wiki

- **The fullest mechanistic account of the CNN [[feature-hierarchy]].** Later features are *assembled* from earlier ones through readable weights: curves from lines, circles from curves, a dog head from eyes, snout and fur, a car from wheels and windows.
- **The founding document of the circuits programme.** That programme carried on into transformer [[mechanistic-interpretability]] at [[anthropic]].
- **A link to the 2026 debate.** Co-author Nick Cammarata later argued that LLM neurons are "already clean" ([[goodfire]], 2026). That is the same features-are-readable position as Claim 1, now applied to Qwen3-32B. See [[monosemanticity]] and [[layer-depth-hierarchy-cnn-vs-llm]].

## References

- [[circuits]] · [[feature-hierarchy]] · [[feature-visualization]] · [[polysemanticity]] · [[superposition]] · [[inceptionv1]] · [[chris-olah]] · [[nick-cammarata]]
- [[feature-visualization-olah-2017]] · [[visualizing-understanding-convnets-zeiler-2013]]
