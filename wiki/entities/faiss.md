---
title: FAISS
type: entity
created: 2026-05-13
updated: 2026-11-11
sources:
  - dpr-karpukhin-2020.md
  - e-means-outperforming-k-means-bessa-2026.md
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

## Newer evidence (2026-11-11)

- e-means, an EMA mini-batch [[k-means]] solver from FAIR, ships in Faiss (`contrib/e_means`). It beats Faiss and Faiss-GPU Lloyd at 25 iterations in about 2 epochs, borrows Faiss's dead-cluster splitting, and uses an IVF index over the centroids for assignment at billion scale. ([[e-means-outperforming-k-means-bessa-2026]])
