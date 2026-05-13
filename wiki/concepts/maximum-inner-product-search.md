---
title: Maximum Inner Product Search (MIPS)
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - dpr-karpukhin-2020.md
status: draft
importance: medium
---

# Maximum Inner Product Search (MIPS)

Given a query vector `q` and a set of `M` candidate vectors `{p₁, ..., p_M}`, MIPS is the problem of finding the top-k candidates with largest inner products `qᵀ pᵢ`. It is the inference-time bottleneck for [[dense-retrieval|dense retrieval]] over large corpora (e.g., 21M Wikipedia passages in [[dpr-karpukhin-2020|DPR]]).

Exact MIPS is O(Md) — usually too slow for serving. Approximate-nearest-neighbor (ANN) libraries — **FAISS** (Johnson et al. 2017), HNSW, ScaNN — answer it in roughly O(log M · d) with controllable recall, by building data structures (IVF, HNSW graphs, product quantization).

## References

- [[dpr-karpukhin-2020]]
