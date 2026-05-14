---
title: "Distributed Representations of Words and Phrases and their Compositionality"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - mikolov-skipgram-2013.md
arxiv_id: "1310.4546"
venue: NeurIPS 2013
authors:
  - "[[tomas-mikolov]]"
  - "[[ilya-sutskever]]"
  - Kai Chen
  - Greg Corrado
  - Jeffrey Dean
first_author: "[[tomas-mikolov]]"
introduces:
  - "[[negative-sampling]]"
year: 2013
tags:
  - 2013
status: complete
importance: high
---

# Distributed Representations of Words and Phrases and their Compositionality

**Source**: `sources/mikolov-skipgram-2013.md` (extracted from `mikolov-skipgram-2013.pdf`)
**arXiv**: [1310.4546](https://arxiv.org/abs/1310.4546)
**Venue**: NeurIPS 2013
**Authors**: [[tomas-mikolov]], [[ilya-sutskever]], Kai Chen, Greg Corrado, Jeffrey Dean (Google)

## Summary

Companion paper to "[[efficient-estimation-of-word-representations-mikolov-2013|Efficient Estimation]]" (Mikolov et al., 2013a). Refines the Skip-gram model with several practical improvements that became standard in [[word2vec|word2vec]] deployments and far beyond:

- **Negative sampling** — a simple alternative to hierarchical softmax: for each positive (word, context) pair, sample `k` "negative" pairs from a noise distribution and train the model to score positives high and negatives low via a binary logistic loss. Vastly faster than full-softmax training and produces higher-quality vectors than hierarchical softmax for frequent words. **This algorithm became the workhorse loss for contrastive representation learning** — DPR, SimCLR, CLIP, sentence-transformers, and most subsequent dense-retrieval systems all trace back to it.
- **Subsampling of frequent words**: discard frequent words (e.g., "the", "of") with probability proportional to their frequency. Speeds training 2–10× and improves accuracy on rare-word vectors.
- **Phrase representations**: a simple data-driven method for identifying multi-word phrases ("Air Canada", "New York Times") and treating them as single tokens — the missing piece that addressed Skip-gram's word-level limitation on idiomatic expressions.
- **Compositionality**: vectors of related concepts compose linearly. `vec("Russian") + vec("river") ≈ vec("Volga")`. Demonstrated for thousands of phrase analogies.

The paper is more engineering-tricks-than-new-theory, but those tricks (negative sampling especially) are foundational to modern representation learning. Ilya Sutskever's involvement here is the bridge between word2vec and his later work on [[seq2seq|seq2seq]] (2014) and the founding of [[openai|OpenAI]] (2015).

## Key Points

- **Negative sampling**: NCE-inspired contrastive objective. For each true context word `w_O`, sample `k` (typically 5–20) "negative" context words from `P(w)^(3/4)` (unigram distribution raised to 3/4). Loss: `log σ(v_O · v_I) + Σ_k E_{w_n ~ P_n} log σ(−v_{w_n} · v_I)`.
- **Subsampling frequent words**: word `w_i` is discarded with probability `1 − √(t/f(w_i))` for some threshold `t ≈ 1e-5`. Removes "the/of/and" noise.
- **Phrase identification**: data-driven scoring `score(w_i, w_j) = (count(w_i w_j) − δ) / (count(w_i) · count(w_j))`. Phrases above a threshold get merged into single tokens before re-training.
- **Compositionality** beyond simple analogies: vector arithmetic produces meaningful results for country-capital, currency, person-occupation, and dozens of other relation types.

## Entities Mentioned

- [[tomas-mikolov]] — first author
- [[ilya-sutskever]] — co-author (then at Google; later co-founder of OpenAI)

## Concepts Discussed

- [[word2vec]] — Skip-gram is one of word2vec's two architectures
- [[negative-sampling]] — the contrastive loss introduced here
- [[word-embeddings]] — the broader paradigm

## Notable Quotes

> "We also describe a simple alternative to the hierarchical softmax called negative sampling." — Abstract

> "Motivated by this example, we present a simple method for finding phrases in text, and show that learning good vector representations for millions of phrases is possible." — Abstract

## References

_Original source: `sources/mikolov-skipgram-2013.md`_
