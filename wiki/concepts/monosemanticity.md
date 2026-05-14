---
title: Monosemanticity
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - cunningham-sparse-autoencoders-2023.md
status: draft
importance: medium
---

# Monosemanticity

A unit (neuron, attention head, or [[sparse-autoencoder|SAE]] feature) is **monosemantic** if it activates on **one** semantically coherent concept — the opposite of [[polysemanticity|polysemantic]] units that fire on many unrelated things.

Monosemanticity is the **interpretability goal**: if you can decompose a network into monosemantic features, each one is amenable to a clean human-language description, and the network's computation becomes a story over named components.

## In real networks

- **Neurons are rarely monosemantic** in production LLMs because of [[superposition]].
- **Attention heads are sometimes monosemantic** — induction heads, IOI heads, etc. — but only a fraction.
- **SAE features are typically monosemantic** when the autoencoder is well-trained and sized appropriately. This is the empirical foundation of the modern interpretability program.

## The Anthropic monosemanticity papers

Two key publications in this lineage (not yet ingested as sources):

- **Bricken et al. 2023, "Towards Monosemanticity"** — SAEs recover monosemantic features from a 1-layer Transformer. Proof-of-concept that the approach works.
- **Templeton et al. 2024, "Scaling Monosemanticity"** — same approach applied to Claude 3 Sonnet (a production frontier model). Discovers safety-relevant features (deception, sycophancy, dangerous content) that can be amplified or suppressed for steering.

The [[emotion-concepts-anthropic-2026|Anthropic 2026 emotion-concepts paper]] is a downstream application: emotion concepts as monosemantic-like linear directions in Claude Sonnet 4.5, with causal effects on behavior.

## References

- [[sparse-autoencoders-cunningham-2023]]
