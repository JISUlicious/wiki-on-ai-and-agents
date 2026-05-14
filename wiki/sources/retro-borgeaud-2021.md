---
title: "Improving Language Models by Retrieving from Trillions of Tokens (RETRO)"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - retro-borgeaud-2021.md
arxiv_id: "2112.04426"
venue: ICML 2022
authors:
  - Sebastian Borgeaud
  - Arthur Mensch
  - "[[jordan-hoffmann]]"
  - Trevor Cai
  - Eliza Rutherford
  - Katie Millican
  - George van den Driessche
  - Jean-Baptiste Lespiau
  - Bogdan Damoc
  - Aidan Clark
  - Diego de Las Casas
  - Aurelia Guy
  - Jacob Menick
  - Roman Ring
  - Tom Hennigan
  - Saffron Huang
  - Loren Maggiore
  - Chris Jones
  - Albin Cassirer
  - Andy Brock
  - Michela Paganini
  - Geoffrey Irving
  - "[[oriol-vinyals]]"
  - Simon Osindero
  - Karen Simonyan
  - Jack W. Rae
  - Erich Elsen
  - Laurent Sifre
year: 2021
tags:
  - 2021
status: complete
importance: high
---

# Improving Language Models by Retrieving from Trillions of Tokens (RETRO)

**Source**: `sources/retro-borgeaud-2021.md` (extracted from `RETRO Borgeud 2021.pdf`)
**arXiv**: [2112.04426](https://arxiv.org/abs/2112.04426)
**Venue**: ICML 2022
**Authors**: Sebastian Borgeaud, Arthur Mensch, Jordan Hoffmann ([[deepmind|DeepMind]]) and ~25 colleagues including Oriol Vinyals, Karen Simonyan, Jack Rae, Laurent Sifre

## Summary

RETRO ("Retrieval-Enhanced Transformer") integrates retrieval **into the language model itself**, at every layer, rather than as a separate preprocessing step over the prompt. At training time, each 64-token chunk of every training sequence retrieves its `k` nearest neighbors (by frozen [[bert|BERT]] embedding similarity) from a multi-trillion-token database; a separate Transformer encoder embeds the retrieved chunks; and a **chunked cross-attention** mechanism interleaved into the decoder lets each chunk attend to its corresponding retrieved neighbors. The retrieved text influences predictions throughout the network, not just via prompt concatenation.

The headline result: a 7.5B-parameter RETRO model trained with retrieval over 2T tokens matches the language-modeling performance of GPT-3 and Jurassic-1 (175B / 178B parameters respectively) on the Pile — **25× fewer parameters**. The retrieval gain is roughly constant from 150M up to 7.5B parameters, meaning retrieval is a **complementary axis to parameter scale** rather than a substitute. Increasing retrieval-database size and number of neighbors both monotonically improve performance.

A subsidiary contribution — **RETROfit** — shows that an existing pre-trained Transformer can be cheaply adapted to use retrieval by adding cross-attention layers and fine-tuning, without retraining from scratch.

RETRO is the first major demonstration that retrieval scales meaningfully when both the database and the parametric model are large. It is a direct intellectual successor to [[rag|RAG]] and [[dpr|DPR]], generalizing retrieval from QA-time augmentation to *pre-training-time* augmentation.

## Key Points

- **Retrieve during pre-training**, not just at inference. Each chunk retrieves neighbors and the model is *trained* to use them.
- **Retriever**: frozen BERT-base encoder. No retriever training needed — fixed embeddings, fixed index. The database can be swapped freely.
- **Chunked structure**: input split into 64-token chunks; each chunk's neighbors only attend to the *next* chunk's tokens, preserving causality.
- **Chunked cross-attention (CCA)**: a new attention mechanism that interleaves with the decoder's self-attention. Time complexity linear in retrieved data.
- **Database**: MassiveText, ~5T tokens; training retrieves from a 600B-token subset; evaluation uses the full 1.75T-token version.
- **Results**:
  - 7.5B RETRO ≈ 175B GPT-3 / 178B Jurassic-1 on Pile language modeling. **25× parameter efficiency.**
  - SOTA on Wikitext-103 and the Pile.
  - Retrieval gain ≈ constant across model sizes (150M–7.5B) — a real complementary axis to parameter scale.
  - Performance improves monotonically with database size and number of retrieved neighbors (up to ~40, after which quality degrades).
- **RETROfit**: existing transformers can be adapted to use retrieval at relatively low cost.
- **Test-set leakage analysis**: since the model has direct access to the training set at evaluation time, the paper introduces a 13-gram Jaccard contamination filter and decomposes gains into "neighbor copying" vs. "general knowledge extraction".

## Entities Mentioned

- [[deepmind]] — author affiliation (all authors)
- [[retro]] — the model
- [[bert]] — frozen retriever
- [[gpt-3]] — comparison baseline (matched at 25× fewer params)

## Concepts Discussed

- [[retrieval-augmented-generation]] — RETRO is a deep, training-time variant of this idea
- [[chunked-cross-attention]] — the architectural innovation
- [[dense-retrieval]] — using frozen BERT embeddings
- [[scaling-laws]] — retrieval as an axis orthogonal to parameter scale
- [[autoregressive-language-modeling]] — RETRO is autoregressive, unlike RAG which is seq2seq

## Notable Quotes

> "With a 2 trillion token database, our Retrieval-Enhanced Transformer (Retro) obtains comparable performance to GPT-3 and Jurassic-1 on the Pile, despite using 25× fewer parameters." — Abstract

> "Our work opens up new avenues for improving language models through explicit memory at unprecedented scale." — Abstract

## References

_Original source: `sources/retro-borgeaud-2021.md`_
