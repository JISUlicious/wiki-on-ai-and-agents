---
title: "Efficient Estimation of Word Representations in Vector Space (word2vec)"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - mikolov-word2vec-2013.md
arxiv_id: "1301.3781"
venue: ICLR Workshop 2013 (arxiv preprint)
authors:
  - "[[tomas-mikolov]]"
  - Kai Chen
  - Greg Corrado
  - Jeffrey Dean
first_author: "[[tomas-mikolov]]"
introduces:
  - "[[word2vec]]"
  - "[[word-embeddings]]"
year: 2013
tags:
  - 2013
status: complete
importance: high
---

# Efficient Estimation of Word Representations in Vector Space (word2vec)

**Source**: `sources/mikolov-word2vec-2013.md` (extracted from `mikolov-word2vec-2013.pdf`)
**arXiv**: [1301.3781](https://arxiv.org/abs/1301.3781)
**Venue**: ICLR Workshop 2013
**Authors**: [[tomas-mikolov]], Kai Chen, Greg Corrado, Jeffrey Dean (Google Inc., Mountain View)

## Summary

This paper introduces the two model architectures behind **word2vec**: **Continuous Bag-of-Words (CBOW)** and **Continuous Skip-gram**. Both learn dense, low-dimensional vector representations of words from unlabeled text — no parse trees, no linguistic features, just word-context co-occurrence on billions of words.

The key design choice: replace the expensive non-linear hidden layer of the standard neural network language model (NNLM, Bengio et al. 2003) with a much simpler architecture. CBOW predicts the current word from a small window of surrounding context words (a kind of "fill in the blank"); Skip-gram does the inverse, predicting the surrounding context words from the current word. Both are trained with hierarchical softmax. The resulting embeddings can be learned from 1.6 billion words in **under a day** on a single machine — orders of magnitude faster than prior NNLM-based approaches at comparable or better quality.

The famous result that made word2vec a household name in NLP: the learned embeddings exhibit **linear arithmetic over semantic relationships**. `vec("King") − vec("Man") + vec("Woman") ≈ vec("Queen")`. Similar relationships hold for verb tenses, country/capital pairs, plural/singular pairs, and more. This was the empirical demonstration that distributional semantics could capture surprisingly rich structure with surprisingly simple methods.

word2vec became the substrate from which every later embedding (GloVe, FastText, ELMo, BERT, dense retrieval) descended. It is the conceptual ancestor of the input-embedding layer of every modern Transformer.

## Key Points

- **CBOW** (Continuous Bag-of-Words): predict word `w_t` from sum of context word vectors `(w_{t-k}, ..., w_{t-1}, w_{t+1}, ..., w_{t+k})`. Order of context words ignored ("bag"). Fast to train; works well with frequent words.
- **Skip-gram**: predict context words from the current word. Slower but produces higher-quality vectors for rare words. The Skip-gram objective dominates downstream use.
- **Linear analogies**: `king − man + woman ≈ queen`. The structure emerges without explicit supervision — purely from co-occurrence statistics. (Pennington et al. would offer a theoretical explanation in [[glove|GloVe]] a year later.)
- **Scale**: trained on 1.6B-word Google News corpus in <24 hours; competing models took weeks for lower quality.
- **Vocabulary**: ~1M words with 300-dimensional vectors. Each word becomes a 300-d real-valued point.
- **Test set introduced**: a now-canonical "semantic-syntactic word relationship" test set of ~20k analogies (Athens→Greece :: Tokyo→?).

## Entities Mentioned

- [[tomas-mikolov]] — first author
- [[google-brain]] (then "Google Inc.") — author affiliation

## Concepts Discussed

- [[word2vec]] — the model family this paper introduced
- [[word-embeddings]] — the broader concept
- [[distributional-semantics]] — the underlying theory

## Notable Quotes

> "We observe large improvements in accuracy at much lower computational cost, i.e. it takes less than a day to learn high quality word vectors from a 1.6 billion words data set." — Abstract

> "vector('King') - vector('Man') + vector('Woman') results in a vector that is closest to the vector representation of the word Queen." — §1.1

## References

_Original source: `sources/mikolov-word2vec-2013.md`_
