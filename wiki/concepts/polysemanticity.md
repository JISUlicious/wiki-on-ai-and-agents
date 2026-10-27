---
title: Polysemanticity
type: concept
created: 2026-05-14
updated: 2026-10-27
sources:
  - cunningham-sparse-autoencoders-2023.md
  - zoom-in-circuits-olah-2020.md
status: draft
importance: medium
---

# Polysemanticity

Polysemanticity is the empirical phenomenon that individual neurons in trained Transformers (and other neural networks) **activate on many semantically unrelated concepts** — a single neuron might fire on French text, code comments, named entities, and exclamation marks, all at once. The opposite of [[monosemanticity|monosemanticity]].

Polysemanticity is the central obstacle to neuron-level interpretability. If a neuron fires on N unrelated concepts, you can't form a clean hypothesis about what it computes; ablating it disrupts N different downstream behaviors. The hypothesized cause is **[[superposition]]**: the network packs more features than it has neurons by distributing each feature across multiple neurons.

## Origin in vision models

The canonical example is InceptionV1 unit **4e:55**, which responds to cat faces, car fronts and cat legs. It detects each separately, with no shared feature behind them ([[zoom-in-circuits-olah-2020]]). Zoom In also traced *why* such neurons form: a clean car neuron in mixed4c is spread across dog neurons in the next layer. This is [[superposition]], and it saves neurons. In both CNNs and LLMs, polysemantic units tend to sit in middle-to-late layers ([[layer-depth-hierarchy-cnn-vs-llm]]).

## Resolution: SAEs

The breakthrough fix is to decompose neuron activations into a sparse, overcomplete feature dictionary using **[[sparse-autoencoder|sparse autoencoders]]** ([[sparse-autoencoders-cunningham-2023|Cunningham et al. 2023]]). SAE features are dramatically more monosemantic than neurons, restoring the interpretability that polysemanticity blocked.

## References

- [[sparse-autoencoders-cunningham-2023]]
- [[zoom-in-circuits-olah-2020]]
