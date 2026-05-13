---
title: Mechanistic Interpretability
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - emotion-concepts-anthropic-2026.md
status: complete
importance: high
---

# Mechanistic Interpretability

Mechanistic interpretability ("mech interp") is the research program of **reverse-engineering** trained neural networks to understand the algorithms they implement — at the level of weights, activations, attention patterns, and circuits, rather than at the level of input-output behavior. Where traditional ML interpretability tries to explain *what* a model predicts, mech interp tries to explain *how* the model arrives at those predictions, in concrete computational terms.

The field is closely associated with [[chris-olah|Chris Olah]] and the [[anthropic|Anthropic]] interpretability team (formerly at OpenAI); foundational papers include the **Distill** "Circuits" thread (2020+) and Anthropic's **Transformer Circuits Thread** (2021+).

## Core methods

- **Activation analysis**: examine what features are encoded in intermediate activations, where, and how they compose.
- **Linear probing / steering vectors**: extract a direction in activation space corresponding to a concept (e.g., [[functional-emotions|"happy"]], "truthfulness," "code"); read out by inner product or write in by adding the vector to activations.
- **Logit lens**: project intermediate-layer activations through the unembedding matrix to see which output tokens they "promote."
- **Attention pattern analysis**: examine where heads attend on specific examples.
- **Circuit discovery**: identify subgraphs of (head, MLP, feature) components that jointly implement an algorithm (e.g., indirect-object identification, modular arithmetic).
- **Sparse autoencoders (SAEs)**: decompose dense activations into sparse "monosemantic" feature dictionaries (Cunningham et al. 2023; Anthropic's "Scaling Monosemanticity" 2024; many follow-ups).
- **Causal mediation / activation patching**: replace one model's activations with another's to determine which components cause a given output.

## Why it matters

- **Trust and safety**: a model whose internal computation we understand is one we can debug, audit, and predict.
- **Alignment**: many alignment failure modes (deception, reward hacking, sandbagging) may be detectable only by inspecting internals.
- **Capability research**: understanding *how* models do what they do informs how to make them do it better — or differently.

## Notable findings

- Linear concept representations are pervasive (the **linear representation hypothesis**).
- Transformer attention heads implement identifiable algorithms (indirect-object identification, induction heads).
- Models can simulate gradient descent in-context (Akyürek et al. 2022; Garg et al. 2022).
- Sparse autoencoders can extract millions of interpretable features from a frontier model's activations.
- Emotion concepts are linearly represented and causally drive behavior, including misaligned behavior ([[emotion-concepts-anthropic-2026|Sofroniew et al. 2026]]).

## Limitations

- Scales poorly with model size — many techniques developed for small models don't trivially extend.
- "Interpretability" is partial; we don't claim full understanding of any frontier model.
- Risk of post-hoc storytelling — a feature labeled "fear" may correlate with fear-related text but actually compute something subtly different.

## References

- [[emotion-concepts-anthropic-2026]]
