---
title: Chunked Cross-Attention
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - retro-borgeaud-2021.md
status: draft
importance: medium
---

# Chunked Cross-Attention

Chunked cross-attention (CCA) is the attention mechanism introduced in [[retro-borgeaud-2021|RETRO]] (Borgeaud et al. 2021) to integrate **retrieved neighbor chunks** into an autoregressive language model while preserving causal generation.

## Setup

The input sequence is split into chunks of size `m` (default 64 tokens). For each input chunk `Cᵤ`, a frozen [[bert|BERT]]-based retriever returns `k` neighbor chunks from a large database. A separate small Transformer encoder embeds the neighbors. CCA layers — interleaved with standard self-attention layers in the decoder — let each input position attend to the encoded neighbors.

## Preserving causality

The crucial detail: neighbors retrieved for chunk `Cᵤ` are only allowed to influence tokens in chunk `Cᵤ₊₁` (and subsequent positions). This way, when generating token `xᵢ`, the model only sees neighbors retrieved from chunks strictly before `xᵢ`'s chunk — no information from the future leaks into the present.

## Compute scaling

CCA's time complexity is **linear** in the amount of retrieved text — the same neighbors are re-attended at every layer but encoded only once. This is what makes retrieving from trillions of tokens at training time feasible.

## References

- [[retro-borgeaud-2021]]
