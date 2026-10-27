---
title: Goodfire
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - neuron-explorer-lyramakesmusic-2026.md
  - neuron-explorer-reactions-2026-10.md
tags:
  - org
status: draft
importance: low
---

# Goodfire

Goodfire is an AI interpretability company. In this wiki it appears through two contributions to [[neuron-explorer]]:

- **Steering fork of [[vllm]]** (RhizoNymph/vllm). It supports [[steering]] per request through a table of steering rows and a per-token index, and it is compatible with CUDA graphs. neuron-explorer runs its sweeps on it.
- **"Neural Networks Are Already Clean"** (Nick Cammarata, 2026-10-15). The essay argues that most neurons are already readable once each neuron's **positive and negative poles** are treated as separate units. Its methods include separating SwiGLU sign regimes, using raw activation scales, reading long context, steering, and a "blind maximum" test with two agents. A gallery of 1,000 written-up Qwen3-32B neuron profiles accompanies it.

> [!note] Source status
> - The essay is not ingested. Its headline numbers reach the wiki only through press coverage: about 90% of Qwen3-32B neuron poles have one main job, about 7% have two related jobs, and 2–4% are unexplained.
> - Those numbers are preliminary, and Cammarata calls part of the argument "highly speculative".
> - Neel Nanda (Google DeepMind) reportedly said the claim stretches "readable" too far.
> - The disagreement with the [[superposition]] view is recorded on [[monosemanticity]].

## References

- [[neuron-explorer-lyramakesmusic-2026]]
