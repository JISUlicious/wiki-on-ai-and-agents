---
title: word2vec
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - mikolov-word2vec-2013.md
  - mikolov-skipgram-2013.md
status: complete
importance: high
---

# word2vec

word2vec is the family of model architectures and training tricks introduced in [[efficient-estimation-of-word-representations-mikolov-2013|Mikolov et al. (2013a)]] and refined in [[distributed-representations-of-words-and-phrases-mikolov-2013|Mikolov et al. (2013b)]] for learning [[word-embeddings|word vector representations]] from large unlabeled text corpora.

## Two architectures

- **CBOW (Continuous Bag-of-Words)**: predict the current word from its context window. The input is the sum (or average) of context word vectors; the model predicts the target word via softmax (or its approximations). Order of context words is ignored.
- **Skip-gram**: predict surrounding context words from the current word. Slower per-iteration but learns better vectors for rare words and dominates in practical use.

Both architectures have only a linear projection and an output layer — no non-linear hidden layer. This is the design choice that made training on billion-word corpora feasible in 2013.

## Training tricks

- **Hierarchical softmax**: replaces the O(|V|) softmax with a binary tree of O(log |V|) decisions. Used in the original word2vec; faster than full softmax.
- **[[negative-sampling]]**: the more impactful alternative from the second paper. For each true (word, context) pair, sample `k` "negative" context words and frame training as a binary classification. Foundational for all later contrastive representation learning.
- **Subsampling of frequent words**: probabilistically drop high-frequency words ("the", "of") during training; speeds training and improves rare-word vectors.
- **Phrase identification**: pre-pass that merges high-PMI bigrams ("New_York") into single tokens.

## What the vectors look like

- Typical: 300 dimensions, ~1M-word vocabulary trained on 1.6B-word Google News.
- **Linear analogies** are the famous result: `vec("Paris") − vec("France") + vec("Italy") ≈ vec("Rome")`.
- Similarity is measured by cosine. The embedding space encodes syntactic regularities (singular/plural, verb tenses) and semantic ones (country/capital, person/occupation) along separable directions.

## Why it mattered

word2vec made dense embeddings a default input layer for NLP systems from 2013–2018 (before contextualized embeddings from ELMo/BERT replaced them). It also introduced [[negative-sampling|negative-sampling]] — the contrastive learning objective that powers DPR, CLIP, sentence-transformers, and the modern retrieval stack.

The input embedding layers of [[gpt-1|GPT-1]], [[bert]], [[gpt-3]], and every modern Transformer are direct lineal descendants — they're learned-from-scratch versions of the same idea, with the rest of the model providing context-sensitivity.

## References

- [[efficient-estimation-of-word-representations-mikolov-2013]]
- [[distributed-representations-of-words-and-phrases-mikolov-2013]]
