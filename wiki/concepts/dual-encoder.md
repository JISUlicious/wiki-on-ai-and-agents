---
title: Dual-Encoder
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - dpr-karpukhin-2020.md
status: draft
importance: medium
---

# Dual-Encoder

A dual-encoder (or "twin tower" or "two-tower") architecture is one where two separate networks independently encode two inputs — typically a query and a candidate document — into vectors of the same dimensionality. The match is computed by a cheap operation on the two vectors (inner product, cosine similarity).

## Why the two-tower split

Because the candidate encoding is **input-independent**, all candidates can be pre-encoded once into a vector index. At query time only the query needs to be encoded; nearest-neighbor search over the pre-built index gives top-k results in milliseconds even over millions of candidates.

This is the architectural property that makes scalable [[dense-retrieval|dense retrieval]] possible. By contrast, a **cross-encoder** that jointly encodes (query, candidate) is more accurate but requires a forward pass per candidate — only feasible for re-ranking small candidate lists.

## Canonical example

[[dpr|DPR]] (Karpukhin et al., 2020) is the prototypical dual-encoder for retrieval: two independent [[bert|BERT]] towers, dot-product similarity, contrastive training.

The dual-encoder pattern long predates DPR (Siamese networks, Bromley et al. 1994; sentence embedding models like InferSent, Sentence-BERT) but DPR's combination of BERT + supervised contrastive learning + MIPS index became the modern standard.

## References

- [[dpr-karpukhin-2020]]
