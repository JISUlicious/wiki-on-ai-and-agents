---
title: Superposition
type: concept
created: 2026-05-14
updated: 2026-10-27
sources:
  - cunningham-sparse-autoencoders-2023.md
  - neuron-explorer-lyramakesmusic-2026.md
  - zoom-in-circuits-olah-2020.md
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

- **First mechanistic sighting, vision (2020).** In [[zoom-in-circuits-olah-2020|Zoom In]], a pure car neuron in InceptionV1 (mixed4c) is spread across dog-detecting neurons in the next layer. The authors' explanation: high-dimensional spaces allow only *n* orthogonal vectors but "exponentially many almost orthogonal vectors".
- **Toy models** (Elhage 2022): controlled experiments confirm small networks adopt superposition under sparsity pressure.
- **SAE recoveries** ([[sparse-autoencoders-cunningham-2023|Cunningham 2023]], Anthropic 2023–2024): in real LLMs, SAEs recover thousands of clean monosemantic features from polysemantic neuron activations.
- **Counterpoint (2026):** Goodfire's "already clean" essay argues most neuron *poles* are readable; see the contradiction note on [[monosemanticity]]. [[neuron-explorer-lyramakesmusic-2026]] finds Qwen3.5-4B's output weights almost orthogonal (9,216 columns in 2,560 dimensions), with thin cluster structure. It also finds mid-layer neurons barely more distinctive than random directions, which fits features spread across many neurons there.
- **Causal probing**: SAE features predict behavior changes more precisely than neurons (e.g., the IOI task experiments in Cunningham et al.).

## References

- [[sparse-autoencoders-cunningham-2023]]
- [[neuron-explorer-lyramakesmusic-2026]]
- [[zoom-in-circuits-olah-2020]]
