---
title: Vector Database
type: concept
created: 2026-11-01
updated: 2026-11-11
sources:
  - vector-database-pinecone.md
  - e-means-outperforming-k-means-bessa-2026.md
status: draft
importance: medium
---

# Vector Database

A store for embeddings searched by similarity, the common retrieval back end in [[rag|RAG]]. See [[vector-database-pinecone]] and [[pinecone]]. This page rests on a vendor source.

## Newer evidence (2026-11-11)

- Building the IVF partition is a [[k-means]] job. e-means reaches Lloyd's 25-epoch loss 10–45× faster on GPU, and clusters 10B DINO vectors into 16.7M clusters in 22.8 GPU-hours. ([[e-means-outperforming-k-means-bessa-2026]])
