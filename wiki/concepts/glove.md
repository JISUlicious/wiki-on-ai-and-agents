---
title: GloVe (Global Vectors)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - pennington-glove-2014.md
status: complete
importance: medium
---

# GloVe (Global Vectors)

GloVe is a method for learning [[word-embeddings|word embeddings]] introduced by [[glove-pennington-2014|Pennington, Socher, & Manning (2014)]] at Stanford. Unlike [[word2vec]]'s local-context prediction approach, GloVe trains a **weighted least-squares regression** over the global word-word co-occurrence matrix:

```
J = Σ_{i,j} f(X_{ij}) (w_i · ŵ_j + b_i + b̂_j − log X_{ij})²
```

where `X_{ij}` is the count of times word `j` appears in the context of word `i`, and `f(x)` is a weighting function that suppresses very rare co-occurrences and caps very common ones.

## word2vec vs. GloVe — practical comparison

| | [[word2vec]] (Skip-gram + NS) | GloVe |
|---|---|---|
| Training signal | Local context windows (predictive) | Global co-occurrence matrix (count-based) |
| Loss | Negative sampling (logistic) | Weighted least squares |
| Memory | Streams over the corpus | Builds the co-occurrence matrix first |
| Quality on analogies | High | Comparable or slightly better |
| Quality on similarity | High | Comparable |
| Speed | Faster for very large corpora | Faster once matrix is built |

In practice, both methods produce similar-quality vectors, and the choice between them was historically driven by available implementations and pre-trained downloads rather than fundamentals.

## Pre-trained GloVe vectors

Stanford released several pre-trained sets at the time of publication:

- **6B**: Wikipedia 2014 + Gigaword 5 (~6B tokens, 400k vocab). 50/100/200/300 dims.
- **42B**: Common Crawl (42B tokens, 1.9M vocab). 300 dims.
- **840B**: Common Crawl (840B tokens, 2.2M vocab). 300 dims.
- **27B**: Twitter (27B tweets, 1.2M vocab). 25/50/100/200 dims.

The 300d GloVe vectors became one of the two default starting points (alongside Mikolov's Google News word2vec) for downstream NLP systems from 2014 until contextualized embeddings (ELMo, BERT) replaced them around 2018.

## Theoretical contribution

The paper's argument for *why* linear analogies emerge: ratios of co-occurrence probabilities `P(k|i)/P(k|j)` encode meaningful relationships between word pairs `(i, j)`. The GloVe objective is designed so that the dot product `w_i · w_j` approximates `log(X_{ij})`, which makes those probability ratios manifest as linear directions in the embedding space.

## References

- [[glove-pennington-2014]]
