---
title: Representation Engineering (RepE)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - representation-engineering-zou-2023.md
status: complete
importance: high
---

# Representation Engineering (RepE)

Representation Engineering is the framework introduced by [[representation-engineering-zou-2023|Zou et al. (2023)]] at the [[center-for-ai-safety|Center for AI Safety]] for **reading and writing internal representations** in deep neural networks. RepE places linear directions in activation space at the center of analysis, drawing on cognitive-neuroscience methods (population-level neural representations) rather than the neuron-by-neuron approach of traditional [[mechanistic-interpretability|mech interp]].

## Two operations

RepE proposes a unified vocabulary for two related interventions:

1. **Reading** ("Linear Artificial Tomography", LAT): given a concept, identify the activation direction that represents it. Methods: difference-in-means, linear probes, PCA on contrastive activations. Use this to *detect* whether the model is thinking about / "in" the concept at a given moment.
2. **Controlling**: add or subtract along the identified direction to push the model toward or away from the concept. Same operation as [[activation-addition|ActAdd]] / [[contrastive-activation-addition|CAA]] but unified under a single framework.

## Top-down vs. bottom-up

The taxonomy RepE proposes:

- **Bottom-up** ([[mechanistic-interpretability|mech interp]]): start from neurons / weights, reverse-engineer the algorithm. Slow, painstaking, doesn't scale easily to safety-relevant high-level concepts.
- **Top-down** (RepE): start from the concept you care about (honesty, harmlessness, power-seeking), find its representation, control it. Faster route to actionable safety interventions, at the cost of weaker mechanistic understanding.

RepE positions these as **complementary** — top-down findings can guide bottom-up investigation and vice versa.

## Safety-relevant demos (per the paper)

- **Honesty / hallucination** (related to [[inference-time-intervention|ITI]]).
- **Harmlessness / refusal control**.
- **Power-seeking detection and reduction**.
- **Emotion** representation (anticipating [[emotion-concepts-anthropic-2026|Anthropic's 2026 emotion-concepts paper]]).
- **Bias and fairness**.
- **Memorization detection**.
- **Instruction-following control**.

For each, RepE shows both that the concept is **linearly readable** and that it is **causally controllable** via activation interventions.

## Why RepE matters

- **Unified terminology** for what was a scattered literature (ActAdd, ITI, probes, contrastive methods) — became the standard reference.
- **Safety-flavored**: explicitly positions interpretability as alignment tooling. Many of the demos target alignment-relevant phenomena.
- **Open-source toolkit** (https://github.com/andyzoujm/representation-engineering) lowered the barrier for follow-up work.

## References

- [[representation-engineering-zou-2023]]
- See [[steering]] for unified treatment alongside other methods.
