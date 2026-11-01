---
title: Monosemanticity
type: concept
created: 2026-05-14
updated: 2026-10-29
sources:
  - cunningham-sparse-autoencoders-2023.md
  - neuron-explorer-lyramakesmusic-2026.md
  - zoom-in-circuits-olah-2020.md
  - sparsified-mlps-vs-transcoders-neurons-in-disguise-shabalin-2026.md
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

> [!warning] Contradiction — "neurons are rarely monosemantic" vs "already clean" (2026)
> - **This page's view, from [[superposition]]:** the neuron is the wrong unit.
> - **Cammarata's counter-claim** ([[goodfire]], "Neural Networks Are Already Clean", 2026-10-15; known here only through press coverage):
>   - about 90% of Qwen3-32B neuron **poles** have one main job, about 7% have two related jobs, and 2–4% are unexplained;
>   - the trick is to treat each neuron's positive and negative poles as separate units, use long context, and look at what the neuron writes;
>   - the figures are preliminary.
> - **Lineage:** Cammarata co-authored [[zoom-in-circuits-olah-2020|Zoom In]] (2020), whose first claim was that features, often single neurons, are readable units in vision models. The same essay documented polysemantic neurons. "Already clean" extends that claim to LLM neurons.
> - **Criticism:** Neel Nanda reportedly said the claim stretches "readable" too far.
> - **Mixed evidence from [[neuron-explorer-lyramakesmusic-2026]]:**
>   - Supporting: about 20% of Qwen3.5-4B L22 neurons steer a theme stronger than any random control.
>   - Against: single mid-layer neurons are barely more distinctive than random directions.
> - **Partial support, carefully scoped ([[sparsified-mlps-vs-transcoders-neurons-in-disguise-shabalin-2026]], Anthropic Fellows / Anthropic, 2026-10):**
>   - The paper uses the same sign-split view, one encoder and decoder per neuron sign.
>   - Across 13 open per-layer transcoders (Gemma, Qwen and Llama, 1B–27B, all dense), many [[transcoder]] features are "neurons in disguise". In the Gemma family (1B→27B) they are 6–37% of the dictionary but 58–83% of activations in **late layers**, and the pattern recurs in the other models.
>   - These neuron features carry essentially all of the transcoder's Jacobian match to the MLP (≈0.3, vs ≈0.05 for the rest). They are also most of the features that recur across training seeds.
>   - Neurons look as monosemantic as transcoder features only at their **top activations**. Over the broad activation range they stay polysemantic. They also miss fine-grained concepts such as bigrams (the dictionary feature fires on 75.6% of target bigrams, neurons on about 0%).
>   - **Reading:** this supports "neurons are a stronger unit than assumed" in late layers and at high activation. It does not support "neurons are clean" in general, and it does not overturn [[superposition]].
> - **Status:** both claims are recorded here. Neither is settled in the wiki's sources.

## The Anthropic monosemanticity papers

Two key publications in this lineage (not yet ingested as sources):

- **Bricken et al. 2023, "Towards Monosemanticity"** — SAEs recover monosemantic features from a 1-layer Transformer. Proof-of-concept that the approach works.
- **Templeton et al. 2024, "Scaling Monosemanticity"** — same approach applied to Claude 3 Sonnet (a production frontier model). Discovers safety-relevant features (deception, sycophancy, dangerous content) that can be amplified or suppressed for steering.

The [[emotion-concepts-anthropic-2026|Anthropic 2026 emotion-concepts paper]] is a downstream application: emotion concepts as monosemantic-like linear directions in Claude Sonnet 4.5, with causal effects on behavior.

## References

- [[sparse-autoencoders-cunningham-2023]]
- [[neuron-explorer-lyramakesmusic-2026]] — neuron-level steering evidence; reports the "already clean" claim secondhand
- [[zoom-in-circuits-olah-2020]]
- [[sparsified-mlps-vs-transcoders-neurons-in-disguise-shabalin-2026]]: transcoder features that duplicate single MLP neurons (late layers, 13 open transcoders)
