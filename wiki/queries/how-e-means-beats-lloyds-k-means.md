---
title: "How does e-means work, and why does it beat Lloyd's k-means in a tenth of the epochs?"
type: query
created: 2026-11-11
updated: 2026-11-11
sources:
  - e-means-outperforming-k-means-bessa-2026.md
status: complete
importance: medium
---

# How does e-means work, and why does it beat Lloyd's k-means in a tenth of the epochs?

**Question (2026-11-11):** Explain e-means.

The answer draws on [[e-means-outperforming-k-means-bessa-2026]] (Bessa et al., FAIR at Meta, [arXiv:2611.09501](https://arxiv.org/abs/2611.09501)). Parts marked **(inference)** are not from the paper.

## The problem it solves

[[k-means]] splits N vectors into k groups and gives each group a center, so that every point sits close to its center. It is an offline step in many systems:
- the clusters in a vector search index (IVF in [[faiss]], [[vector-database|vector databases]]);
- [[quantization]] codebooks;
- data curation;
- the topic tree [[retriever-free-retrieval-decision-models-wan-2026|DecisionRAG]] walks.

The standard method is **Lloyd's algorithm**. It assigns every point to its nearest center, recomputes each center as the mean of its points, and repeats. Each round reads the whole dataset, and Faiss runs 25 rounds by default. That gets expensive at 1 billion vectors and 1 million clusters.

## The idea

e-means works on **small random batches** instead of the whole dataset. After each batch it nudges every center toward the batch points assigned to it:

```
for each batch:
    assign batch points to their nearest centroid
    N_c ← γ·N_c + (1−γ)·count_c        # decayed count per cluster
    S_c ← γ·S_c + (1−γ)·sum_c          # decayed sum per cluster
    centroid_c = S_c / N_c
```

Each center is an **exponential moving average** (EMA) of the points it receives. The rule comes from VQ-VAE codebooks. Batch-by-batch k-means (Sculley 2010) already existed, but it improved fast early and then settled at a worse result than Lloyd. e-means adds four things that fix that:

1. **The update size shrinks over training.** The weight on new points (α = 1 − γ) starts at α₀ and falls to 0.1·α₀ on a cosine curve. Big early steps move centers quickly; small late steps let them settle.
2. **Empty clusters get rebuilt.** Every 5 batches, any cluster with fewer than 10% of the average count is replaced by splitting a large cluster in two. This matters most: without it, error rises by 3%.
3. **The final centers are exact means.** At the end, each center is recomputed as the mean of the points assigned to it in the last pass. An alternative ending keeps a second, slower EMA, which does better on points not seen in training.
4. **No settings to tune.** Batch size and starting step size come from fitted formulas in n (points), k (clusters) and T (passes):
   - α₀ = min(0.824·k^−0.46·B^0.90·n^−0.45·T^−0.11, 1)
   - B = 0.035·k^0.13·n^0.88·T^0.81

   They were fitted from about 75,000 runs on one dataset and lose only 0.07–0.19% on other datasets. This works like the [[scaling-laws]] LLM training uses to pick batch size and learning rate.

## Why batches are enough (inference)

Lloyd computes each new center from every point, even though a sample would give nearly the same center. With batches, the centers improve many times per pass over the data, where Lloyd improves them once. The early updates are noisy, but the shrinking step and the exact-mean ending remove most of that noise.

## Results

- **Speed:** it beats Lloyd's error after 25 passes in only **1.4–2.6 passes**. On GPU that is **10–45× faster** than Flash-KMeans, with the biggest gains at 1B points and 1M clusters.
- **Quality:** it ends with lower error than Lloyd on every dataset tested, on both training and held-out data. Usually it doesn't need k-means++ initialization either.
- **Very large scale:** at 10B points and 16.7M clusters, comparing each point with every center is too slow. A search index over the centers finds each point's nearest one instead. That cut the run from 779 to **22.8 GPU-hours**. The 10B-vector set (features from the [[dinov2|DINO]] lineage) is released as a new benchmark, **DINO10B**.

## Limits

- The formulas are tuned for high-dimensional embeddings. They aren't the best choice for very low-dimensional or tiny datasets.
- The authors say what really matters is the data's *intrinsic* dimension, which you usually don't know.
- The search-index speedup only helps at very large scale: 34× cheaper at 10B points, 4.3× at 1B, none at 100M.

## Why it matters for retrieval (inference)

When clustering costs a tenth as much and needs no tuning, rebuilding an index after the embeddings change becomes routine instead of a planned job. The same goes for DecisionRAG's topic tree and quantization codebooks.

## References

- [[e-means-outperforming-k-means-bessa-2026]]
- [[k-means]] · [[faiss]] · [[vector-database]] · [[quantization]] · [[scaling-laws]]
