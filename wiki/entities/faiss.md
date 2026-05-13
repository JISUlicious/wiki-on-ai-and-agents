---
title: FAISS
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - dpr-karpukhin-2020.md
status: draft
importance: low
tags:
  - product
---

# FAISS

FAISS (Facebook AI Similarity Search; Johnson, Douze, & Jégou 2017) is an open-source library from [[fair|Facebook AI]] for efficient similarity search and clustering of dense vectors. It is the de-facto standard for [[maximum-inner-product-search|MIPS]] over millions to billions of vectors via approximate-nearest-neighbor structures (IVF, HNSW, product quantization).

In this wiki, FAISS appears as the index backend used by [[dpr|DPR]] (and almost every other [[dense-retrieval|dense retrieval]] system since).

## References

- [[dpr-karpukhin-2020]]
