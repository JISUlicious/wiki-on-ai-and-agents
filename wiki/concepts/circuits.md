---
title: Circuits (interpretability)
type: concept
created: 2026-10-27
updated: 2026-10-27
sources:
  - zoom-in-circuits-olah-2020.md
status: draft
importance: medium
---

# Circuits (interpretability)

A **circuit** is a subgraph of a neural network: a set of features plus the weights connecting them, studied as a readable algorithm. The term and the research programme come from the Distill Circuits thread on [[inceptionv1|InceptionV1]] ([[zoom-in-circuits-olah-2020]]). Its three founding claims:

1. **Features** are the fundamental unit, and they correspond to directions in activation space.
2. **Circuits** of features are meaningful and can be studied rigorously.
3. **Universality:** analogous features and circuits form across models.

## Canonical vision circuits

| Circuit | Depth | Motif |
|---|---|---|
| Curve detectors built from earlier curve and line detectors ("tangent curves"; opposite orientations inhibit) | 2 layers, early | **equivariance**: the weights rotate with the orientation |
| Left- and right-facing dog-head pathways that inhibit each other, then merge into invariant units | 4 layers, high | **unioning over cases** |
| A pure car neuron (windows above, wheels below) spread across dog neurons in the next layer | mixed4c → next layer | **[[superposition]]** |

## In transformers

The programme moved to language models as transformer circuits work at [[anthropic]]: induction heads, the indirect-object-identification task, and [[sparse-autoencoder|SAE]] features. See [[mechanistic-interpretability]].

The [[neuron-explorer]] "connections" view is a weight-based circuit tool for MLP neurons. It computes direct-path links from each neuron's output weights to the next layers' input weights.

## References

- [[zoom-in-circuits-olah-2020]] · [[feature-visualization-olah-2017]]
- [[feature-hierarchy]] · [[polysemanticity]] · [[superposition]] · [[chris-olah]]
