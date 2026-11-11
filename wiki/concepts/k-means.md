---
title: k-means
type: concept
created: 2026-11-11
updated: 2026-11-11
sources:
  - e-means-outperforming-k-means-bessa-2026.md
  - retriever-free-retrieval-decision-models-wan-2026.md
  - kvquant-10-million-context-kv-quantization-hooper-2024.md
status: draft
importance: medium
---

# k-means

Partition n vectors into k clusters by choosing k centroids that minimise the mean squared distance from each point to its nearest centroid. It is one of the most-used algorithms in ML infrastructure, usually as an offline step:
- **Vector search:** IVF partitions in [[faiss]] and [[vector-database|vector databases]], and product-quantization codebooks.
- **Compression:** non-uniform [[quantization]] codebooks, e.g. SqueezeLLM, and KVQuant's nuqX datatype for [[kv-cache-quantization|KV-cache quantization]].
- **Retrieval structure:** building a corpus hierarchy when none exists, as the recursive k-means tree in [[retriever-free-retrieval-decision-models-wan-2026|DecisionRAG]] does on Natural Questions.
- **Attention and data:** sparse attention (ClusterKV), data curation and deduplication.

## Solvers

| Solver | Update | Notes |
|---|---|---|
| **Lloyd (full batch)** | Assign all points, then set each centroid to the mean of its points | Converges to a local minimum. Usually capped at about 25 iterations (the Faiss default). |
| **k-means++** seeding | Spread-out initial centroids | Better starting point, but the sequential seeding is slow at scale. |
| **Mini-batch (MacQueen 1967, Sculley 2010)** | Running-mean SGD on batches | Fast early, but settles at a higher error than Lloyd. The batch size and step must be guessed. |
| **EMA centroids (VQ-VAE)** | Exponential moving average of assigned points | Built to track moving distributions; never compared with Lloyd until e-means. |
| **e-means (2026)** | EMA + cosine-annealed step + dead-cluster revival + last-epoch-mean finalization; batch and step set by scaling laws | Beats Lloyd's 25-epoch loss in 1.4–2.6 epochs; 10–45× faster on GPU; scales to 10B points and 16.7M clusters with IVF assignment |

Speed-ups that work alongside these solvers:
- bounds that skip distance computations (Elkan, Yinyang, SuperKMeans);
- tuned GPU kernels (Faiss-GPU, Flash-KMeans);
- coresets.

## Recurring problems

- **Dead clusters:** a centroid that stops receiving points stays empty. Reviving it by splitting a large cluster is the single most important part of the e-means recipe.
- **Hyperparameters for mini-batch variants:** e-means fits power laws in (n, k, epochs) for the batch size and step. They transfer across embedding datasets but are tuned for high-dimensional data.
- **Train vs. held-out error:** you can finalize centroids for the training set (exact last-epoch means) or for new vectors (a slow EMA). Which is better depends on the use.

## References

- [[e-means-outperforming-k-means-bessa-2026]]
- [[retriever-free-retrieval-decision-models-wan-2026]]
- [[kvquant-10-million-context-kv-quantization-hooper-2024]]
