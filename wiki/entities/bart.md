---
title: BART
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - rag-lewis-2020.md
family: transformer
predecessor: "[[transformer]]"
parameters: "140M (base) / 400M (large)"
release_date: 2019-10-29
status: draft
importance: medium
tags:
  - model
---

# BART

BART (Bidirectional and Auto-Regressive Transformers; Lewis et al. 2019, arXiv 1910.13461) is a denoising-pre-trained encoder-decoder Transformer from [[fair|Facebook AI]]. It combines [[bert|BERT]]'s bidirectional encoder with a [[gpt-1|GPT]]-style autoregressive decoder, trained to reconstruct corrupted input text (token masking, deletion, sentence permutation, document rotation, text infilling).

BART achieves strong results on summarization, dialogue, and translation. In this wiki, BART appears as the **generator backbone** of [[rag-lewis-2020|RAG]] (BART-large, 400M params).

## References

- [[rag-lewis-2020]]
