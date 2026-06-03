---
title: "The Spike, the Sparse and the Sink: Anatomy of Massive Activations and Attention Sinks"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - massive-activations-attention-sinks-sun-2026.pdf
arxiv_id: "2603.05498"
authors:
  - Shangwen Sun
  - Alfredo Canziani
  - Yann LeCun
  - Jiachen Zhu
first_author: Shangwen Sun
year: 2026
tags: [2026]
status: complete
importance: medium
---

# The Spike, the Sparse and the Sink: Anatomy of Massive Activations and Attention Sinks

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

This mechanistic study examines two phenomena that reliably co-occur in decoder-only, pre-norm Transformer LLMs: massive activations (a few tokens show extreme outlier values in a few channels) and attention sinks (certain tokens attract disproportionate attention mass regardless of semantic relevance). Prior work noted they often involve the same tokens but treated the link descriptively; this paper provides a causal account.

The core finding is that the co-occurrence is not intrinsic to Transformers but a predictable artifact of architectural and training choices — specifically the pre-norm configuration. Massive activations act globally, interacting with normalization to produce near-constant hidden representations that persist across layers and function as implicit parameters. Attention sinks act locally, modulating per-head attention output and biasing heads toward short-range dependencies. Ablating pre-norm decouples the two, and each can be independently suppressed without degrading language-modeling performance.

## Key points

- Massive activations and attention sinks are related but functionally distinct; their overlap is an architectural artifact, not a functional necessity.
- Normalization (pre-norm) is the bridge: massive activations × normalization yield near-constant hidden states that serve as implicit model parameters.
- Attention sinks are driven mainly by attention-space dimensionality and the training context-length distribution.
- Sinks dynamically modulate attention across heads, biasing some heads toward local sentence structure.
- Each phenomenon can be suppressed independently with no loss in LM performance — the co-occurrence is incidental.
- Practical relevance for quantization, pruning, KV-cache management, and long-context inference.

## Concepts & entities

- [[attention-mechanism]]
- [[multi-head-attention]]
- [[layer-normalization]]
- [[decoder-only-transformer]]
- [[transformer-architecture]]
- [[mechanistic-interpretability]]
- [[kv-cache]]
- [[long-context-llm]]

## References

- [arXiv:2603.05498](https://arxiv.org/abs/2603.05498)
