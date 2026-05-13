---
title: Dense Retrieval
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - dpr-karpukhin-2020.md
status: complete
importance: high
---

# Dense Retrieval

Dense retrieval uses **learned, low-dimensional vector representations** of queries and documents (typically a few hundred dimensions from a neural encoder) and retrieves nearest neighbors by [[maximum-inner-product-search|maximum inner product]] (or cosine) similarity. It is the modern alternative to **sparse retrieval** ([[bm25|BM25]], TF-IDF), which represents queries and documents as high-dimensional sparse vectors over a vocabulary.

## Why dense?

- **Semantic matching**: dense embeddings can match synonyms and paraphrases ("bad guy" ↔ "villain") that share no exact tokens.
- **Learnable**: representations can be fine-tuned on labeled (query, document) pairs to adapt to a task or domain.
- **Composable with neural readers**: dense embeddings naturally feed into downstream neural systems.

## Why sparse still matters

- **No training data required**: BM25 works out of the box.
- **Strong on exact-match queries** (names, dates, identifiers, code, jargon).
- **Cheap to update**: adding new documents requires no encoder forward pass.

In practice, hybrid retrieval (BM25 + dense, often reranked) is common.

## Canonical recipe

The dominant dense-retrieval recipe was crystallized by [[dpr-karpukhin-2020|DPR]] (Karpukhin et al., 2020): a [[dual-encoder|dual-encoder]] (independent [[bert|BERT]] towers for query and document), contrastive training with in-batch negatives + one hard BM25 negative, dot-product similarity, FAISS index for serving.

## Variants

- **Cross-encoder reranking**: a single BERT scoring (query, doc) jointly — much more accurate but too slow to run over millions of docs; used to rerank top-k from a dense or sparse first stage.
- **Late-interaction models** (ColBERT, Khattab & Zaharia 2020): per-token embeddings instead of one vector per doc, scored via MaxSim. Better quality at the cost of larger indices.
- **Generative retrieval** (DSI, NCI): generate document IDs directly from a seq2seq model.
- **Hybrid retrieval**: combine dense and sparse scores at the document level.

Dense retrieval is the retriever component of most modern [[rag|RAG]] systems.

## References

- [[dpr-karpukhin-2020]]
