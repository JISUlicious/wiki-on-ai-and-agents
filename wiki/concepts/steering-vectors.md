---
title: Steering Vectors
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - turner-activation-engineering-2023.md
  - contrastive-activation-addition-rimsky-2023.md
status: draft
importance: medium
---

# Steering Vectors

A steering vector is a **direction in a Transformer's activation space** that encodes a target concept (sentiment, truthfulness, emotion, a behavior) and can be added to the residual stream during inference to push the model toward that concept. The fundamental artifact of [[steering|steering]].

Where the vector comes from varies by method:

- [[activation-addition|ActAdd]]: a single contrastive prompt-pair difference.
- [[contrastive-activation-addition|CAA]]: average of differences over hundreds of contrastive examples.
- [[representation-engineering|RepE]]: linear probes / PCA on a labeled concept dataset.
- [[sparse-autoencoder|SAE]] features: unsupervised — autoencoder decoder rows give per-feature steering directions.
- [[inference-time-intervention|ITI]]: per-attention-head probe directions.

## Properties

- **Layer-specific**: the same concept is encoded differently at different layers. Middle layers are usually most useful for behavioral steering.
- **Approximately linear**: the [[linear-representation-hypothesis|LRH]] is the assumption that concepts compose linearly. Composition mostly works for related concepts; mixed steers can interfere unpredictably.
- **Tunable strength**: applying `c · v` with `c` ∈ ℝ controls how strongly the model is pushed. Negative `c` suppresses the concept.

## References

- [[activation-engineering-turner-2023]]
- [[contrastive-activation-addition-rimsky-2023]]
- See [[steering]] for the unified treatment.
