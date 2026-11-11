---
title: "e-means: outperforming k-means (in one-tenth the epochs)"
type: source
created: 2026-11-11
updated: 2026-11-11
sources:
  - e-means-outperforming-k-means-bessa-2026.md
arxiv_id: "2611.09501"
authors:
  - Swann Bessa
  - Pierre-Emmanuel Mazaré
  - Gergely Szilvasy
  - Hugo Touvron
  - David Picard
  - Matthijs Douze
  - Hervé Jégou
introduces:
  - "[[k-means]]"
year: 2026
venue: Preprint (arXiv cs.LG)
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# e-means: outperforming k-means (in one-tenth the epochs)

**Source:** Swann Bessa, Pierre-Emmanuel Mazaré, Gergely Szilvasy, Hugo Touvron, David Picard, Matthijs Douze, Hervé Jégou. Affiliations: [[fair|FAIR]] at [[meta|Meta]]; École Nationale des Ponts et Chaussées. [arXiv:2611.09501](https://arxiv.org/abs/2611.09501), cs.LG (also cs.AI), v1 6 Nov 2026. Code is in [[faiss|Faiss]] (`contrib/e_means`). Website: e-means.github.io.

## Summary

[[k-means]] clustering is everywhere in retrieval and ML infrastructure: vector-index partitioning, quantization codebooks, data curation, and in LLM stacks for KV-cache compression and sparse attention. The standard solver is Lloyd's full-batch algorithm, run for about 25 passes over the data (the Faiss default).

**e-means** replaces Lloyd with mini-batch updates in which each centroid is an **exponential moving average** (EMA) of the points assigned to it. The EMA rule comes from VQ-VAE codebooks, where it was used to track a *moving* distribution. Here it is adapted to the static k-means problem and paired with **scaling laws that set the batch size and step size from (n, k, epochs)**, so no hyperparameter sweep is needed.

On embedding datasets from 1M to 10B vectors, it **beats Lloyd's loss at 25 epochs in 1.4–2.6 epochs**. That is a 10–45× GPU wall-clock speedup over Flash-KMeans and Faiss-GPU, and it converges to a lower loss. It often beats Lloyd seeded with k-means++.

## Method (Algorithm 1)

- **EMA statistics.** For each cluster, keep a decayed count N and sum S: `N ← γN + (1−γ)n_batch` and `S ← γS + (1−γ)s_batch`. The centroid is S/N and is never stored. Initialization sets N = 10⁻¹⁰, so the random starting centroids carry almost no weight.
- **Cosine-annealed step.** The step α = 1 − γ follows a cosine schedule from α₀ down to 0.1·α₀. It is new in this setting and gives small gains.
- **Dead-cluster revival.** Every 5 batches, clusters whose count drops below 10% of the mean are rebuilt by splitting a donor cluster, the same procedure Faiss uses. This is the most important component: removing it costs +3.0% train MSE. VQ-VAE's resampling from data is worse (+2.1%).
- **Finalization, two options:**
  - **Last-epoch mean:** reuse the last epoch's assignments to recompute exact means. This lowers train MSE at almost no cost and is the default.
  - **Slow EMA:** a second, slower EMA of the centroids, which generalizes better to held-out vectors.
- **Initialization.** Random points from the data. k-means++ helps slightly but costs more than simply training longer.

## Scaling laws for hyperparameters

Both laws are fitted by least squares in log space on one dataset (DEEP), from about 75k runs:
- **Peak step:** α₀ = min(0.824 · k^−0.46 · B^0.90 · n^−0.45 · T^−0.11, 1).
- **Batch size:** B = 0.035 · k^0.13 · n^0.88 · T^0.81, the largest batch within 0.2% of the best MSE. It is rounded to a power of two and divided by 8 when n > 2M.

The laws transfer to other datasets with 0.07–0.19% regret (SIFT, SPACEV, and arXiv and AG-News from VIBE). They are tuned for high-dimensional data and are not optimal on very low-dimensional or tiny datasets. The authors note the real driver is the data's *intrinsic* dimension, which is usually unknown.

## Results

**Epochs to match Flash-KMeans at 25 epochs, and wall-clock speedup (Table 4):**

| n | k | Epochs needed | Speedup vs Flash-KMeans |
|---|---|---|---|
| 1M | 16,384 | 1.4–2.4 | 9.5–12.9× |
| 10M | 65,536 | 1.4–2.0 | 19.2–27.8× |
| 100M | 262,144 | 1.4–2.6 | 19.9–35.9× |
| 1B | 1,048,576 | 1.4–2.0 | 21.2–45.5× |

- **Against Lloyd:** ends below it on every dataset, on both train and test MSE. Earlier mini-batch methods (Sculley 2010, Staleness-Reduction) beat Lloyd after one epoch but settle at a higher MSE.
- **Against k-means++:** with random init, 3 epochs of e-means beat Flash-KMeans at 25 iterations on all 8 configurations. Against Flash-KMeans seeded with k-means++, e-means still wins after 5 epochs, by 0.06–0.33%.
- **On CPU:** faster than Faiss k-means and the distance-pruning method SuperKMeans.
- **At extreme scale, approximate assignment.** Each point's nearest centroid is found with an IVF index over the centroids:
  - **DINO10B (10B vectors, 16.7M clusters):** 22.8 GPU-hours instead of 779, 34× cheaper.
  - **1B vectors:** 4.3× cheaper.
  - **100M vectors:** no gain.
- **DINO10B:** a new dataset of 10 billion DINOv3 features (1024 dimensions), released as a benchmark for clustering at scale and for approximate nearest-neighbour search.
- **Implementation:** CuTe GPU kernels with BF16/FP8/INT8 assignment, in blocks that never materialise the full B×k distance matrix.

## Why it matters for this wiki

k-means is the offline step behind many retrieval systems. Examples:
- IVF partitions in [[faiss]] and [[vector-database|vector databases]];
- the recursive k-means corpus tree in [[retriever-free-retrieval-decision-models-wan-2026|DecisionRAG]] (1,000 clusters over 108k Wikipedia pages);
- quantization codebooks ([[quantization]], [[kvquant-10-million-context-kv-quantization-hooper-2024|KVQuant]]'s nuqX datatype);
- the clustering used in data curation.

A 10× cheaper solver with no tuning makes reclustering large embedding stores routine. The paper's method of fitting [[scaling-laws|scaling laws]] for hyperparameters echoes how LLM training sets batch size and learning rate.

## Concepts & entities

- [[k-means]]: the problem and the e-means solver.
- [[faiss]] · [[fair]] · [[meta]] · [[dinov2]] (the DINO lineage behind DINO10B).
- [[vector-database]] · [[quantization]] · [[kv-cache-quantization]] · [[scaling-laws]].

## References

- [arXiv:2611.09501](https://arxiv.org/abs/2611.09501)
