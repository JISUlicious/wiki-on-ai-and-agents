---
title: Activation Engineering
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - turner-activation-engineering-2023.md
status: complete
importance: medium
---

# Activation Engineering

Activation engineering is the broader name [[alexander-matt-turner|Turner et al.]] gave to **inference-time modification of model activations to steer outputs**. Introduced in [[activation-engineering-turner-2023|Turner et al. (2023)]] alongside the specific algorithm [[activation-addition]] (ActAdd).

The framing: where prompt engineering modifies the **input tokens** to steer a model, activation engineering modifies the **hidden activations** during the forward pass. Same goal (control behavior); different surface.

For the broader practice — including [[representation-engineering|RepE]], [[contrastive-activation-addition|CAA]], [[inference-time-intervention|ITI]], and [[sparse-autoencoder|SAE]]-based interventions — the wiki uses **[[steering]]** as the umbrella term. "Activation engineering" is functionally a synonym, with slightly more emphasis on the inference-time-intervention aspect.

## Why it works

The implicit assumption is the **[[linear-representation-hypothesis|linear representation hypothesis]]**: concepts are encoded as roughly linear directions in activation space. Adding a vector along a "concept direction" produces a corresponding behavior change without disrupting unrelated computations.

This is empirically robust across model sizes (Llama 2, Llama 3, OPT, Claude, GPT scale) and concepts (sentiment, toxicity, truthfulness, power-seeking, emotion, etc.).

## References

- [[activation-engineering-turner-2023]]
- See [[steering]] for the unified treatment.
