---
title: Sparse Autoencoder (SAE)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - cunningham-sparse-autoencoders-2023.md
builds-on: "[[mechanistic-interpretability]]"
status: complete
importance: high
---

# Sparse Autoencoder (SAE)

A sparse autoencoder is a neural-network-based **dictionary learning** tool applied to language-model activations: given the residual-stream activations of a Transformer at a given layer, learn an **overcomplete sparse code** that decomposes the activations into a (sparse, additive) combination of **monosemantic features**. Each learned feature corresponds to a narrow, interpretable concept; together they form a "feature dictionary" for the model.

Introduced in the LLM-interpretability context by [[sparse-autoencoders-cunningham-2023|Cunningham et al. (2023, ICLR 2024)]] at [[eleutherai|EleutherAI]] and the Bristol AI Safety Centre. Anthropic dramatically scaled the approach with Towards Monosemanticity (Bricken et al. 2023) and Scaling Monosemanticity (Templeton et al. 2024) on Claude 3 Sonnet.

## Why SAEs exist: superposition

The motivating problem is **[[polysemanticity|polysemanticity]]** — individual neurons in trained Transformers activate in many semantically unrelated contexts. The hypothesized cause is **[[superposition]]**: the network encodes more features than it has neurons, distributing each as a (sparse, near-orthogonal) direction in activation space rather than as a single neuron.

If superposition is real, then interpreting neurons directly is doomed: each neuron is a sum of many features. You need to *decompose* the activations into the underlying features.

## Architecture and training

```
                encoder           sparsity-penalty          decoder
activations ──────────────► hidden_code ─────────► reconstructed_activations
   (d-dim)            (k-dim, k >> d, ~10x typical)            (d-dim)
```

- **Overcomplete**: hidden dim `k` much larger than input dim `d` (typically 8×–64×).
- **Sparsity penalty**: L1 on the hidden code (or top-k masking). Forces most of the `k` features to be near-zero on each input.
- **Reconstruction loss**: standard squared-error on the autoencoder output.
- **Result**: each hidden unit becomes a **monosemantic feature detector** that fires on a narrow, interpretable concept.

## What SAE features look like

From Anthropic's Scaling Monosemanticity on Claude 3 Sonnet:

- "Famous bridges" feature
- "Sycophantic agreement" feature
- "Lying / deception" feature
- "Code injection" feature
- "Dangerous biological content" feature
- Multilingual / multimodal features (same feature fires on the same concept in different languages, or on text and images)

The 34M-feature SAE has features for surprisingly specific concepts — concepts the model is *demonstrably tracking* internally but that aren't obvious from outputs alone.

## Why SAEs matter for steering

Once you have an SAE, **every feature is a steering direction**. The encoder gives a *readable* signal ("how much is the model thinking about deception right now?"); amplifying or suppressing a feature gives a *writable* steering intervention — at the level of interpretable concepts rather than hand-picked contrastive prompts.

This is the modern foundation of the [[steering]] toolkit. [[representation-engineering|RepE]]-style probes need labeled contrastive data; SAE features are *unsupervised*, automatically discovered, and exist in their thousands for any production model.

## Open problems

- **Feature labeling at scale**: with millions of features, automated labeling (via probing or auto-interp) becomes essential. Quality is improving but imperfect.
- **Completeness**: SAEs miss features ("missing features" / "feature gaps"). Reconstruction error indicates how much the SAE is failing to capture.
- **Layer choice**: features at different layers represent different abstraction levels (token-level → syntactic → semantic → meta-cognitive). Choosing which layer to SAE matters.
- **Causal validation**: a feature might *correlate* with a concept but not *cause* outputs related to it. Causal probing remains active research.

## References

- [[sparse-autoencoders-cunningham-2023]] — the foundational LLM-SAE paper
- See [[steering]] and [[mechanistic-interpretability]] for the broader contexts.
- Anthropic's Towards Monosemanticity (Oct 2023) and Scaling Monosemanticity (May 2024) — the production-scale SAE work; not yet ingested.
