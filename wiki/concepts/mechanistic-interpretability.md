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

- The [[linear-representation-hypothesis|linear representation hypothesis]] — concepts as linear directions in activation space — is pervasive and the operating assumption of the modern interpretability program.
- Transformer attention heads implement identifiable algorithms (indirect-object identification, induction heads).
- Models can simulate gradient descent in-context (Akyürek et al. 2022; Garg et al. 2022).
- **[[sparse-autoencoder|Sparse autoencoders]]** ([[sparse-autoencoders-cunningham-2023|Cunningham 2023]], Bricken 2023, Templeton 2024) can extract thousands-to-millions of interpretable features from a frontier model's activations, addressing [[polysemanticity]] and [[superposition]].
- Emotion concepts are linearly represented and causally drive behavior, including misaligned behavior ([[emotion-concepts-anthropic-2026|Sofroniew et al. 2026]]).

## The relationship to steering

Mech interp's interventions (ablations, activation patching, feature suppression) are mechanistically the same operation as **[[steering]]**: change something in the residual stream, observe behavior change. The fields differ in framing:

- Mech interp asks *how* the network computes — bottom-up from neurons / heads / circuits.
- [[representation-engineering|RepE]] / steering asks *what* to push the network toward — top-down from high-level concepts.

In practice the toolkits converge: [[sparse-autoencoder|SAEs]] give mech interp its preferred decomposition primitive, and the resulting features are by-construction usable as steering directions. [[activation-addition|ActAdd]], [[contrastive-activation-addition|CAA]], and [[inference-time-intervention|ITI]] are all bottom-up enough to count as mech-interp interventions and top-down enough to count as RepE / steering.

## Limitations

- Scales poorly with model size — many techniques developed for small models don't trivially extend (though SAE-based methods are catching up; Anthropic's Scaling Monosemanticity on Claude 3 Sonnet is a recent SOTA).
- "Interpretability" is partial; we don't claim full understanding of any frontier model.
- Risk of post-hoc storytelling — a feature labeled "fear" may correlate with fear-related text but actually compute something subtly different.

## References

- [[emotion-concepts-anthropic-2026]]
- [[sparse-autoencoders-cunningham-2023]]
- [[representation-engineering-zou-2023]]
- [[inference-time-intervention-li-2023]]
