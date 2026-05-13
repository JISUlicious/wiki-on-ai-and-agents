---
title: "Generating Wikipedia by Summarizing Long Sequences"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-decoder-liu-2018.md
arxiv_id: "1801.10198"
venue: ICLR 2018
authors:
  - Peter J. Liu
  - Mohammad Saleh
  - Etienne Pot
  - Ben Goodrich
  - Ryan Sepassi
  - Łukasz Kaiser
  - Noam Shazeer
year: 2018
tags:
  - 2018
status: complete
importance: medium
---

# Generating Wikipedia by Summarizing Long Sequences

**Source**: `sources/transformer-decoder-liu-2018.md` (extracted from `Transformer Decoder Liu 1801.10198.pdf`)
**arXiv**: [1801.10198](https://arxiv.org/abs/1801.10198)
**Venue**: ICLR 2018
**Authors**: Peter J. Liu, Mohammad Saleh, Etienne Pot, Ben Goodrich, Ryan Sepassi, Łukasz Kaiser, [[noam-shazeer]] ([[google-brain]])

## Summary

Frames English Wikipedia generation as **multi-document summarization**: given a Wikipedia topic title and a collection of non-Wikipedia reference documents, generate the lead section (or full article) of the Wikipedia entry. The authors use extractive summarization to coarsely pre-select salient content from the reference documents, then an abstractive neural model to write the article.

The paper's lasting architectural contribution — and the reason it appears in this wiki — is a **[[decoder-only-transformer|decoder-only Transformer]] variant** ("T-DMCA": Transformer-Decoder with Memory-Compressed Attention) that scales to very long inputs by treating the input + output as a single token sequence and using causal self-attention throughout, rather than an explicit encoder-decoder split. This is directly cited by [[gpt-1-radford-2018|GPT-1]] (as reference [34]) as the architectural inspiration for using a "multi-layer Transformer decoder" as a language model.

The paper shows the decoder-only variant outperforms RNN seq2seq and standard encoder-decoder [[transformer-architecture|Transformers]] on this long-input summarization task and can generate "fluent, coherent multi-sentence paragraphs and even whole Wikipedia articles."

## Key Points

- **Task formulation**: Wikipedia generation as multi-document summarization with extractive pre-filtering + abstractive generation.
- **Architectural contribution**: a decoder-only Transformer that handles longer inputs better than encoder-decoder Transformers.
- **Memory-compressed attention**: a variant designed to handle long inputs by compressing keys/values, reducing attention cost from O(n²) toward something tractable.
- **Empirically**: outperforms RNN seq2seq and encoder-decoder Transformer baselines on Wikipedia lead generation (ROUGE, human eval).
- **Historical importance**: cited by [[gpt-1-radford-2018|GPT-1]] as the source of the "Transformer decoder" architecture used for language modeling. Without this paper's decoder-only formalization, the GPT line might have taken a slightly different shape.

## Entities Mentioned

- [[noam-shazeer]] — co-author (also co-authored the original [[transformer|Transformer paper]])
- [[google-brain]] — author affiliation

## Concepts Discussed

- [[decoder-only-transformer]] — the architectural pattern this paper crystallized
- [[transformer-architecture]] — modified into decoder-only form
- Memory-compressed attention (specific to this paper, not yet a wiki concept)

## Notable Quotes

> "For the abstractive model, we introduce a decoder-only architecture that can scalably attend to very long sequences, much longer than typical encoder-decoder architectures used in sequence transduction." — Abstract

## References

_Original source: `sources/transformer-decoder-liu-2018.md`_
