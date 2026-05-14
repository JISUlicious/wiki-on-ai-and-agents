---
title: "GloVe: Global Vectors for Word Representation"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - pennington-glove-2014.md
arxiv_id: null
venue: EMNLP 2014
authors:
  - "[[jeffrey-pennington]]"
  - "[[richard-socher]]"
  - "[[christopher-manning]]"
first_author: "[[jeffrey-pennington]]"
introduces:
  - "[[glove]]"
year: 2014
tags:
  - 2014
status: complete
importance: high
---

# GloVe: Global Vectors for Word Representation

**Source**: `sources/pennington-glove-2014.md` (extracted from `pennington-glove-2014.pdf`)
**Venue**: EMNLP 2014
**Paper**: https://aclanthology.org/D14-1162/
**Authors**: [[jeffrey-pennington]], [[richard-socher]], [[christopher-manning]] ([[stanford-university|Stanford NLP]])

## Summary

GloVe (Global Vectors) is an alternative to [[word2vec]] for learning [[word-embeddings|word embeddings]] that combines the two main pre-existing families of methods:

- **Global matrix factorization** (LSA-style): efficiently exploits the global co-occurrence statistics of a corpus.
- **Local context-window** methods (word2vec-style): captures linear analogy structure but trains only on local windows.

GloVe is a **weighted least squares regression** over the (logged) word-word co-occurrence matrix. The objective trains each pair `(i, j)` to satisfy `w_i · w_j ≈ log(X_{ij})`, where `X_{ij}` is the count of times word `j` appeared in the context of word `i`. A weighting function `f(X_{ij})` suppresses noise from very rare pairs and saturates for very common ones.

Empirically, GloVe matched or exceeded Skip-gram + Negative Sampling on the word-analogy benchmark introduced by [[efficient-estimation-of-word-representations-mikolov-2013|Mikolov et al. (2013)]] — 75% on the analogy test set — and on word-similarity benchmarks and NER. The 300-dimensional pre-trained GloVe vectors from Stanford (released alongside the paper) became one of the two default off-the-shelf word embedding choices through 2017 (the other being Mikolov's Google News word2vec vectors), used as input layers for countless downstream NLP systems.

The paper's theoretical contribution: it derives *why* the linear-analogy structure (`king − man + woman ≈ queen`) emerges. The argument is that ratios of co-occurrence probabilities encode meaningful relationships, and the GloVe objective is designed precisely to make `w_i · w_j` track those log-ratios.

## Key Points

- **Objective**: `J = Σ_{i,j} f(X_{ij}) (w_i^T ŵ_j + b_i + b̂_j − log X_{ij})²` where `X_{ij}` is the co-occurrence count, `w_i` and `ŵ_j` are word and context vectors, and `f` weights pairs by frequency.
- **Weighting function** `f(x) = min(1, (x/x_max)^α)` with `x_max = 100` and `α = 0.75`. Caps the influence of very common co-occurrences.
- **Why log ratios**: ratios like `P(solid|ice)/P(solid|steam)` are large for words like "solid" that are meaningful contrasts between "ice" and "steam", and ~1 for irrelevant words like "fashion". The model is designed so vector dot products approximate these log-ratios.
- **Pre-trained vectors released**: 50d/100d/200d/300d trained on Common Crawl (840B tokens, 2.2M vocab) and Wikipedia + Gigaword (6B tokens, 400k vocab). Both became canonical inputs to downstream NLP systems.
- **75% on analogy benchmark** — matches or beats Skip-gram + NS at the time.

## Entities Mentioned

- [[jeffrey-pennington]] — first author
- [[richard-socher]] — co-author; later founded MetaMind (acquired by Salesforce)
- [[christopher-manning]] — Stanford NLP director; one of the most influential NLP educators of the era
- [[stanford-university]] — author affiliation

## Concepts Discussed

- [[glove]] — the method itself
- [[word-embeddings]] / [[word2vec]] — the broader paradigm and contrast point

## Notable Quotes

> "We analyze and make explicit the model properties needed for such regularities to emerge in word vectors. The result is a new global log-bilinear regression model that combines the advantages of the two major model families in the literature: global matrix factorization and local context window methods." — Abstract

## References

_Original source: `sources/pennington-glove-2014.md`_
