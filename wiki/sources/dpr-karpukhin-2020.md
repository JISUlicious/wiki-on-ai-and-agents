---
title: "Dense Passage Retrieval for Open-Domain Question Answering"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - dpr-karpukhin-2020.md
arxiv_id: "2004.04906"
venue: EMNLP 2020
authors:
  - Vladimir Karpukhin
  - Barlas Oğuz
  - Sewon Min
  - Patrick Lewis
  - Ledell Wu
  - Sergey Edunov
  - Danqi Chen
  - Wen-tau Yih
year: 2020
tags:
  - 2020
status: complete
importance: high
---

# Dense Passage Retrieval for Open-Domain Question Answering

**Source**: `sources/dpr-karpukhin-2020.md` (extracted from `DPR Karpuhkin 2020 2004.04906.pdf`)
**arXiv**: [2004.04906](https://arxiv.org/abs/2004.04906)
**Venue**: EMNLP 2020
**Authors**: Vladimir Karpukhin, Barlas Oğuz, Sewon Min ([[university-of-washington|UW]]), [[patrick-lewis]], Ledell Wu, Sergey Edunov, [[danqi-chen]] ([[princeton-university|Princeton]]), Wen-tau Yih ([[fair|Facebook AI]])

## Summary

DPR introduces a **dual-encoder dense-retrieval system** for [[open-domain-qa|open-domain question answering]] that uses two [[bert|BERT]]-base encoders — one for questions, one for passages — trained end-to-end to maximize the inner product of (question, gold passage) pairs while minimizing it for irrelevant passages. At inference, all corpus passages are pre-encoded into a vector index; for a new question, the question encoder produces a query vector and [[maximum-inner-product-search|maximum inner product search (MIPS)]] retrieves the top-k passages from millions in milliseconds.

The paper's punchline is empirical: a simple supervised dual-encoder with only ~1k–60k labeled question-passage pairs **substantially outperforms** the long-standing sparse-retrieval baseline [[bm25|BM25]] (e.g., 65.2% vs 42.9% top-5 retrieval accuracy on Natural Questions), and the gain propagates to end-to-end QA (41.5% vs 33.3% exact match). The earlier ORQA system (Lee et al., 2019) had shown dense retrieval *could* outperform BM25, but only via a heavy "inverse cloze task" pre-training; DPR shows that pre-training-free dual-encoder fine-tuning on QA pairs is sufficient — simpler and stronger.

DPR became the canonical dense-retrieval recipe and is a direct dependency of [[rag-lewis-2020|RAG]] (Lewis et al., 2020, same group) and of [[retro-borgeaud-2021|RETRO]] (DeepMind, 2021).

## Key Points

- **Architecture**: dual encoder. `EQ(q)` and `EP(p)` are independent BERT-base towers (768-d). Similarity = dot product.
- **Training**: contrastive — for each (question, positive passage) pair, treat other in-batch passages plus 1 BM25-hard negative as negatives. Cross-entropy loss over the batch.
- **Inference**: passages pre-encoded offline; questions encoded on the fly; FAISS-based MIPS over ~21M Wikipedia passages.
- **Why it beats BM25**: dense embeddings handle synonyms and paraphrases that share no exact tokens (e.g., "bad guy" ↔ "villain"). They are also *learnable* — fine-tuning adapts the representation to the task.
- **Why supervised dual-encoder beats ORQA's heavy pre-training**: question-passage labeled pairs already contain rich signal; the encoders just need to learn what "relevant" looks like, not learn language from scratch.

## Entities Mentioned

- [[patrick-lewis]] — co-author; will lead [[rag-lewis-2020|RAG]] the same year
- [[danqi-chen]] — co-author; well-known QA researcher at [[princeton-university|Princeton]]
- [[fair]] — Facebook AI Research (most authors' affiliation)
- [[university-of-washington]] — Sewon Min
- [[princeton-university]] — Danqi Chen

## Concepts Discussed

- [[dense-retrieval]] — the central idea
- [[dual-encoder]] — the architectural pattern
- [[bm25]] — the sparse baseline being displaced
- [[maximum-inner-product-search]] — the inference-time retrieval algorithm
- [[open-domain-qa]] — the task setting
- [[bert]] — the encoder backbone

## Notable Quotes

> "Our final solution is surprisingly simple: the embedding is optimized for maximizing inner products of the question and relevant passage vectors, with an objective comparing all pairs of questions and passages in a batch." — §1

> "Our Dense Passage Retriever (DPR) is exceptionally strong. It not only outperforms BM25 by a large margin (65.2% vs. 42.9% in Top-5 accuracy), but also results in a substantial improvement on the end-to-end QA accuracy compared to ORQA (41.5% vs. 33.3%)." — §1

## References

_Original source: `sources/dpr-karpukhin-2020.md`_
_Code: https://github.com/facebookresearch/DPR_
