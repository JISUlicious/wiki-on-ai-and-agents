---
title: RETRO (Retrieval-Enhanced Transformer)
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - retro-borgeaud-2021.md
family: retrieval-augmented
predecessor: "[[rag]]"
successors: []
parameters: "172M / 425M / 1.5B / 7.5B"
release_date: 2021-12-08
status: complete
importance: high
tags:
  - model
---

# RETRO (Retrieval-Enhanced Transformer)

RETRO is the retrieval-augmented autoregressive language model introduced in "[[retro-borgeaud-2021|Improving Language Models by Retrieving from Trillions of Tokens]]" (Borgeaud et al., DeepMind, ICML 2022). It is the first system to integrate retrieval **into pre-training** at scale, retrieving from a multi-trillion-token database.

## Architecture

- **Base**: standard decoder-only Transformer LM (with prepended RETRO blocks).
- **Chunking**: input sequence split into chunks of 64 tokens.
- **Retriever**: frozen pre-trained [[bert|BERT]]-base encoder. Each chunk's BERT embedding is used as a query into a [[maximum-inner-product-search|MIPS]] index over the database; top-`k` (default k=2) neighbor chunks of ~5 tokens each are retrieved.
- **Neighbor encoder**: a small Transformer encoder embeds the retrieved chunks.
- **Chunked cross-attention (CCA)**: a new attention layer interleaved with standard self-attention. For each input chunk, attends to its retrieved neighbors. Causality is preserved by having chunk `i`'s neighbors only affect chunk `i+1`'s tokens.

| Variant | Params (non-embed) | Layers |
|---|---|---|
| RETRO 172M | 172M | — |
| RETRO 425M | 425M | — |
| RETRO 1.5B | 1.5B | — |
| RETRO 7.5B | 7.5B | — |

## Training

- **Corpus**: MassiveText (~5T tokens, multi-lingual). Retrieve from a 600B-token subset during training.
- **Evaluation database**: 1.75T tokens (full MassiveText minus a books subsample, with 13-gram contamination filtering against test sets).
- **Objective**: standard [[autoregressive-language-modeling|next-token prediction]] — the retrieval-augmented decoder learns when and how to use retrieved context.

## Key contribution: retrieval as an orthogonal scaling axis

- 7.5B RETRO matches 175B [[gpt-3]] / 178B Jurassic-1 on the Pile.
- Retrieval gain is **constant across model sizes** (150M–7.5B parameters): retrieval is a complementary axis, not a substitute, for parameter scaling.
- Database-size and number-of-neighbors scaling also yield monotonic improvements.

## RETROfit

A separately-proposed procedure: take an existing pre-trained Transformer LM, add retrieval cross-attention layers, fine-tune. Recovers most of RETRO's gains at a fraction of the training cost.

## Lineage

| Predecessor | Successor |
|---|---|
| [[rag]] (Lewis et al. 2020) — seq2seq retrieval-augmented generation | Future training-time-retrieval systems |

## References

- [[retro-borgeaud-2021]]
