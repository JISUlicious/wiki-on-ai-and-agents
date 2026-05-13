---
title: Transformer Decoder (T-DMCA)
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-decoder-liu-2018.md
family: transformer
predecessor: "[[transformer]]"
successors:
  - "[[gpt-1]]"
release_date: 2018-01-30
status: complete
importance: medium
tags:
  - model
---

# Transformer Decoder (T-DMCA)

The Transformer-Decoder (sometimes called "T-DMCA", Transformer-Decoder with Memory-Compressed Attention) is the decoder-only variant of the [[transformer]] introduced in [[transformer-decoder-liu-2018|"Generating Wikipedia by Summarizing Long Sequences"]] (Liu et al., ICLR 2018) for long-input multi-document summarization.

Its primary significance for this wiki is **architectural lineage**: this paper is the cited inspiration for the decoder-only Transformer architecture used by [[gpt-1]] and, by extension, the entire [[gpt-family|GPT family]] of language models.

## Key idea

Rather than an encoder-decoder split, the model treats the (very long) input documents + the target article as a single token sequence and applies a single decoder stack with causal self-attention end-to-end. To handle the long input efficiently, it introduces **memory-compressed attention** — strided convolutions over keys and values to reduce the effective sequence length seen by attention.

See [[decoder-only-transformer]] for the broader concept (which is what was widely adopted; the specific T-DMCA memory-compression scheme was not).

## Lineage

| Predecessor | Successor |
|---|---|
| [[transformer]] (Vaswani 2017) | [[gpt-1]] (Radford 2018) |

## References

- [[transformer-decoder-liu-2018]]
