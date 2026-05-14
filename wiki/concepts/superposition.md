---
title: Superposition
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - cunningham-sparse-autoencoders-2023.md
status: draft
importance: medium
---

# Superposition

Superposition is the hypothesis that a neural network **encodes more features than it has neurons** by representing each feature as a (sparse, near-orthogonal) direction in activation space rather than as a single dedicated neuron. Articulated in detail in Elhage et al. (2022) "Toy Models of Superposition" (Anthropic) and broadly accepted as the leading explanation for [[polysemanticity|polysemanticity]] in real Transformers.

The intuition: in high dimensions, you can pack many almost-orthogonal directions into a single vector space. If features are sparsely activated (only a few features active per input), the network can "use" the same neurons to encode different features at different times, with relatively little interference. This is more parameter-efficient than dedicating one neuron per feature — explaining why SGD finds this regime.

## Why it matters

If superposition is real, the neuron is the wrong unit of analysis for interpretability. The right unit is a **direction in activation space**. This shift is the methodological foundation of:

- [[sparse-autoencoder|Sparse autoencoders]] — explicitly decompose activations into a sparse, overcomplete *feature* basis rather than analyzing neurons.
- [[steering]] — operate on directions, not neurons.
- [[representation-engineering|RepE]] — linear-direction probes for concepts.

## Evidence

- **Toy models** (Elhage 2022): controlled experiments confirm small networks adopt superposition under sparsity pressure.
- **SAE recoveries** ([[sparse-autoencoders-cunningham-2023|Cunningham 2023]], Anthropic 2023–2024): in real LLMs, SAEs recover thousands of clean monosemantic features from polysemantic neuron activations.
- **Causal probing**: SAE features predict behavior changes more precisely than neurons (e.g., the IOI task experiments in Cunningham et al.).

## References

- [[sparse-autoencoders-cunningham-2023]]
