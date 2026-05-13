---
title: DPR (Dense Passage Retriever)
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - dpr-karpukhin-2020.md
family: retrieval
predecessor: null
successors:
  - "[[rag]]"
release_date: 2020-04-10
status: complete
importance: high
tags:
  - model
---

# DPR (Dense Passage Retriever)

DPR is the dual-encoder dense-retrieval system introduced in "[[dpr-karpukhin-2020|Dense Passage Retrieval for Open-Domain Question Answering]]" (Karpukhin et al., EMNLP 2020) by [[fair|Facebook AI]] + [[university-of-washington|UW]] + [[princeton-university|Princeton]] collaborators.

## Architecture

Two independent [[bert|BERT-base]] encoders:

- `EQ(q)` — question encoder, 768-d output (the `[CLS]` representation).
- `EP(p)` — passage encoder, 768-d output (the `[CLS]` representation).
- Similarity score: `sim(q, p) = EQ(q)ᵀ EP(p)`.

## Training

In-batch negative contrastive learning:

- For each labeled (question, positive passage) pair in a batch of size B, treat the other B-1 passages in the batch as in-batch negatives.
- Add one [[bm25|BM25]]-hard negative per example.
- Cross-entropy loss over the (B + 1) candidates.

Trained on combined open-domain QA datasets (Natural Questions, TriviaQA, WebQuestions, CuratedTREC, SQuAD).

## Inference

- All ~21M Wikipedia passages (100-token chunks) encoded once, stored in a [[faiss|FAISS]]-style vector index.
- Question encoded at query time.
- [[maximum-inner-product-search|MIPS]] retrieves top-k passages (typically k=20–100).
- Top-k passed to an extractive reader (a fine-tuned BERT) for span extraction.

## Results

- Top-5 retrieval accuracy (Natural Questions): **65.2%** vs. BM25's 42.9%.
- Open-domain QA exact match (NQ): **41.5%** vs. ORQA's 33.3%.

## Lineage / influence

DPR is the canonical dense-retrieval recipe. Its direct successors / dependents:

- [[rag]] (Lewis et al., 2020) — wraps DPR-style retrieval around a generative seq2seq (BART) for non-extractive QA and generation.
- [[retro]] (Borgeaud et al., 2021) — DeepMind's retrieval-augmented LM, scaling the idea to language modeling rather than QA.

The dual-encoder + MIPS pattern is now standard in semantic search, ColBERT-style retrieval, and modern RAG systems.

## References

- [[dpr-karpukhin-2020]]
