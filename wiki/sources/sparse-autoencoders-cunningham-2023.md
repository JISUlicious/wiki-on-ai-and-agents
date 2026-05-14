---
title: "Sparse Autoencoders Find Highly Interpretable Features in Language Models"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - cunningham-sparse-autoencoders-2023.md
arxiv_id: "2309.08600"
venue: ICLR 2024 (arxiv preprint Sep 2023)
authors:
  - Hoagy Cunningham
  - Aidan Ewart
  - Logan Riggs
  - Robert Huben
  - Lee Sharkey
first_author: Hoagy Cunningham
introduces:
  - "[[sparse-autoencoder]]"
  - "[[superposition]]"
  - "[[polysemanticity]]"
  - "[[monosemanticity]]"
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# Sparse Autoencoders Find Highly Interpretable Features in Language Models

**Source**: `sources/cunningham-sparse-autoencoders-2023.md`
**arXiv**: [2309.08600](https://arxiv.org/abs/2309.08600)
**Venue**: ICLR 2024
**Authors**: Hoagy Cunningham, Aidan Ewart, Logan Riggs (equal first), Robert Huben, Lee Sharkey ([[eleutherai|EleutherAI]], MATS, Bristol AI Safety Centre, Apollo Research)

## Summary

This paper introduces **[[sparse-autoencoder|sparse autoencoders (SAEs)]]** as a scalable, unsupervised tool for resolving **[[superposition]]** and **[[polysemanticity]]** in language model activations. The problem: individual neurons in transformer language models tend to be polysemantic — a single neuron activates in many semantically unrelated contexts, making the network's internal representations opaque. The hypothesized cause is *superposition*: the network encodes more features than it has neurons, distributing each feature across multiple neurons as a (sparse, nearly orthogonal) direction in activation space.

The fix: train an autoencoder on the LM's residual-stream activations with a sparsity penalty (L1 on the hidden code). The autoencoder's hidden units become **monosemantic feature detectors** — each one activates on a narrow, interpretable concept (e.g., "first-person pronouns in scientific writing," "the letter A in a particular position," "comparison phrases"). Where individual neurons are messy, SAE features are clean.

The paper demonstrates: (a) SAE features are more interpretable than neurons or PCA-based decompositions by automated interpretability metrics; (b) SAE features identify causally important directions — ablating an SAE feature predicts behavior changes on the indirect-object-identification task more precisely than ablating neurons does.

This paper is the **foundation of the SAE-based interpretability program** that Anthropic later scaled dramatically with Towards Monosemanticity (Bricken et al. 2023) and Scaling Monosemanticity (Templeton et al. 2024, on Claude 3 Sonnet). The [[emotion-concepts-anthropic-2026|2026 emotion-concepts paper]] is in this lineage — though it uses linear-probe-style steering vectors rather than SAE features.

## Key Points

- **Problem**: polysemanticity (one neuron = many concepts) blocks interpretability. Hypothesized cause: superposition (more features than neurons → features encoded as directions, not neurons).
- **SAE solution**: train an autoencoder `f(x) = decoder(top-k(encoder(x)))` on residual-stream activations, with L1 sparsity penalty. Hidden code dim much larger than input dim (overcomplete). Force most code units to be near-zero per input.
- **Result**: hidden units of the SAE correspond to clean, interpretable features. Automated interpretability (Bills et al. 2023) scores them more highly than neurons.
- **Causal validation**: on the indirect-object-identification (IOI) task, ablating SAE features pinpoints the causally responsible directions more precisely than ablating individual neurons does.
- **Scalability**: SAEs are simple — single matrix multiply each direction, plus a sparsity penalty — and the technique scales linearly with model size. This is why it became Anthropic's preferred interpretability primitive.
- **Foundation for safety-relevant steering**: once you have clean features, you can amplify or suppress them via [[activation-addition|activation addition]] just like with [[representation-engineering|RepE]] probes — but with fine-grained, automatically-discovered concepts instead of hand-chosen contrastive pairs.

## Entities Mentioned

- Hoagy Cunningham, Aidan Ewart, Logan Riggs — equal-first co-authors
- [[eleutherai]] — primary affiliation
- Apollo Research — Lee Sharkey's affiliation

## Concepts Discussed

- [[sparse-autoencoder]] — the technique
- [[polysemanticity]] — the problem
- [[superposition]] — the hypothesized cause
- [[monosemanticity]] — the goal
- [[mechanistic-interpretability]] — the parent field
- [[steering]] — SAEs enable feature-level steering

## Notable Quotes

> "One of the roadblocks to a better understanding of neural networks' internals is polysemanticity, where neurons appear to activate in multiple, semantically distinct contexts. ... One hypothesised cause of polysemanticity is superposition, where neural networks represent more features than they have neurons by assigning features to an over-complete set of directions in activation space, rather than to individual neurons." — Abstract

> "Our method may serve as a foundation for future mechanistic interpretability work, which we hope will enable greater model transparency and steerability." — Abstract

## References

_Original source: `sources/cunningham-sparse-autoencoders-2023.md`_
