---
title: "Steering Language Models With Activation Engineering"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - turner-activation-engineering-2023.md
arxiv_id: "2308.10248"
venue: arxiv preprint (Aug 2023; v5 Oct 2024)
authors:
  - "[[alexander-matt-turner]]"
  - Lisa Thiergart
  - Gavin Leech
  - David Udell
  - Juan J. Vazquez
  - Ulisse Mini
  - Monte MacDiarmid
first_author: "[[alexander-matt-turner]]"
introduces:
  - "[[activation-addition]]"
  - "[[activation-engineering]]"
  - "[[steering-vectors]]"
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# Steering Language Models With Activation Engineering

**Source**: `sources/turner-activation-engineering-2023.md`
**arXiv**: [2308.10248](https://arxiv.org/abs/2308.10248)
**Venue**: arxiv preprint (Aug 2023; updated through Oct 2024)
**Authors**: [[alexander-matt-turner]] (independent), Lisa Thiergart (MIRI), Gavin Leech (Bristol), David Udell (independent), Juan J. Vazquez (Arb Research), Ulisse Mini (MATS), Monte MacDiarmid ([[anthropic]])

## Summary

This paper formalizes **activation engineering** — modifying a Transformer's activations **at inference time** to steer its outputs, without any optimization, fine-tuning, or prompt modification. It introduces the **Activation Addition (ActAdd)** technique: compute a [[steering-vectors|steering vector]] as the difference between a model's intermediate activations on two contrastive prompts (e.g., `"Love"` minus `"Hate"`) at a chosen layer, then **add that vector to the residual stream** during a real forward pass on a different input. The model's generation shifts in the direction of the steering vector — toward "more loving," "less toxic," etc.

The paper's central claim is the "elicitation overhang": LLMs harbor capabilities that prompt engineering can't fully unlock, but activation engineering can. ActAdd achieves SOTA on negative-to-positive sentiment shift and detoxification using LLaMA-3 and OPT, while preserving performance on unrelated tasks.

ActAdd is the foundational reference for the modern **[[steering|steering]]** literature. It demonstrated that linear interventions in the residual stream produce predictable, controllable, off-target-preserving behavior changes. Every later steering method ([[contrastive-activation-addition|CAA]], [[representation-engineering|RepE]], SAE-based steering) is a refinement of the ActAdd idea.

## Key Points

- **ActAdd algorithm**: pick a contrastive prompt pair (`p+`, `p-`); record residual-stream activations at a chosen layer for both; compute the difference vector `h_+ − h_−`; **add this vector** (scaled by a coefficient `c`) to the residual stream of every forward pass on test inputs.
- **No training required**. Single pair of demonstrations sufficient. **Computationally trivial**.
- **Layer choice matters**: typically middle layers (8–18 in a 32-layer model) work best.
- **Strength tunable**: coefficient `c` controls how strongly to push. Too small = no effect; too large = degraded fluency.
- **Linear representation hypothesis** is the implicit theoretical foundation: concepts like "loving" vs. "hating" are encoded as approximately linear directions in activation space, so arithmetic operations on those directions produce predictable effects.
- **Empirical results**: SOTA on sentiment shift, detoxification on multiple base models. Off-target performance (on Wikipedia perplexity, ARC-Challenge, etc.) is preserved within noise.
- **Lightweight relative to alternatives**: prompt engineering is unreliable; RLHF is expensive; ActAdd is one vector subtraction per inference.

## Entities Mentioned

- [[alexander-matt-turner]] — first author; central figure in the steering literature
- Monte MacDiarmid — Anthropic co-author
- [[anthropic]], MIRI, MATS — author affiliations

## Concepts Discussed

- [[activation-engineering]] — the broader paradigm
- [[activation-addition]] — the specific algorithm
- [[steering-vectors]] — the artifact
- [[steering]] — the meta-concept
- [[linear-representation-hypothesis]] — implicit theoretical basis
- [[mechanistic-interpretability]] — sibling field

## Notable Quotes

> "We introduce activation engineering: the inference-time modification of activations in order to control (or steer) model outputs. Specifically, we introduce the Activation Addition (ActAdd) technique, which contrasts the intermediate activations on prompt pairs (such as 'Love' versus 'Hate') to compute a steering vector." — Abstract

> "ActAdd yields inference-time control over high-level output properties (like topic and sentiment) while preserving performance on off-target tasks. ActAdd is lightweight: it does not require any machine optimization and works with a single pair of data points." — Abstract

## References

_Original source: `sources/turner-activation-engineering-2023.md`_
