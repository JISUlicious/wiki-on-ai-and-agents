---
title: "Rethinking Knowledge Retrieval for Generation: A Survey on RAG Architectures and Applications"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - rethinking-knowledge-retrieval-rag-survey-sunil-2026.md
arxiv_id: "2610.01936"
authors:
  - Meghana Sunil
  - Shravya V
  - Shravan Venkatraman
  - Joe Dhanith P R
year: 2026
status: complete
importance: low
tags:
  - 2026
  - paper
---

# Rethinking Knowledge Retrieval for Generation: A Survey on RAG Architectures and Applications

**Meghana Sunil, Shravya V, Shravan Venkatraman, Joe Dhanith P R** (School of Computer Science and Engineering, Vellore Institute of Technology, Chennai). [arXiv:2610.01936](https://arxiv.org/abs/2610.01936), v2, cs.AI, 6 Oct 2026 (v1 1 Oct). Preprint survey with about 380 references.

## Summary

This is a broad survey of [[retrieval-augmented-generation]]. It breaks RAG into three parts: **retrieval** (dense and sparse retrievers, embeddings), **generation** (fusing the retrieved context with the query) and **augmentation** (indexing, fine-tuning and embedding refinements). It recaps the familiar progression Naïve RAG → Advanced RAG → Modular RAG. The organizing contribution is a taxonomy along four axes:

1. **Compression and efficiency:** billion-document retrieval, few-shot retrieval, and trading curated training data against external corpora.
2. **Defensive RAG:** bias, adversarial or misleading context, privacy leakage.
3. **Interactive, user-centric RAG:** personalization and conversational systems.
4. **Complex reasoning:** multi-step and multi-hop retrieval.

## Key points

- **Applications** are surveyed by domain: QA, security and fairness, chatbots, code generation, education and enterprise, scientific and legal text. Tables collect reported results per model.
- **Open problems:** retrieval bottlenecks at scale and under domain shift; chunk-size trade-offs; hallucination despite retrieval; domain adaptation; latency; explainability.
- **Future directions:** RL-trained retrieval policies, neuro-symbolic RAG, multimodal and structured retrieval, better benchmarks.

## What it does not show

- **No new experiments.** Tables restate numbers reported by other papers.
- **Dated literature.** Its arXiv citations stop around May 2025 (2505.x), despite a 2026 date. It misses the 2025–26 shift to agentic or reasoning-time retrieval that the wiki tracks elsewhere ([[agentic-search]], [[search-r1]], [[when-to-retrieve-during-reasoning-guo-2026]]).
- The newsletter blurb ("survey on RAG architectures") is accurate. There are no numbers to check.

## Concepts & entities

- [[retrieval-augmented-generation]] · [[rag]] · [[rag-lewis-2020]]: the paradigm and its origin.
- [[dense-retrieval]] · [[bm25]] · [[dpr]] · [[hallucination]] · [[knowledge-graph]] · [[hipporag-2]]
- [[agentic-search]]: the direction the survey under-covers.

## References

- [arXiv:2610.01936](https://arxiv.org/abs/2610.01936)
