---
title: Word Embeddings
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - mikolov-word2vec-2013.md
  - mikolov-skipgram-2013.md
status: complete
importance: high
---

# Word Embeddings

Word embeddings are dense, low-dimensional vector representations of words learned from text data. Each word is mapped to a vector in some `d`-dimensional real-valued space (typically `d = 50–300` historically, much larger in modern transformer embeddings). Words with similar distributional behavior end up near each other in the embedding space.

The breakthrough in 2013 was demonstrating that **distributional semantics** — Firth's "you shall know a word by the company it keeps" — could be operationalized as a tractable learning problem. [[efficient-estimation-of-word-representations-mikolov-2013|Mikolov et al. (2013)]] showed dense embeddings could be learned from billions of words on a single machine and would exhibit surprisingly rich structure: linear analogies (`king − man + woman ≈ queen`), clustering by part of speech, robust transfer to downstream tasks.

## Canonical families

- **[[word2vec]]** (Mikolov et al. 2013): predict word from context (CBOW) or context from word (Skip-gram). Trained with [[negative-sampling]] or hierarchical softmax.
- **[[glove|GloVe]]** (Pennington et al. 2014): factorize the global word-word co-occurrence matrix via weighted least squares. An algebraic alternative to word2vec's online prediction objective.
- **FastText** (Bojanowski et al. 2016): word2vec at the character-n-gram level — handles out-of-vocabulary words and morphology.

All three produce **static** embeddings: one vector per word type, regardless of context. The next wave — **contextual embeddings** — gives each word *instance* its own vector based on surrounding sentence:

- **ELMo** (Peters et al. 2018): contextualized embeddings from a bi-LSTM language model.
- **[[bert|BERT]]** (Devlin et al. 2018): contextualized embeddings from a pre-trained Transformer encoder.

By 2019, contextual embeddings dominated NLP tasks, but static word vectors remained useful as cheap inputs to downstream systems and as the conceptual gateway from "words as discrete symbols" to "words as points in continuous space."

## Why this mattered for the LLM trajectory

Word embeddings were the first proof-of-concept that **continuous, learned representations of language are tractable and transferable**. Every later representation — sentence embeddings, [[dense-retrieval|dense passage retrieval]], multimodal alignments (CLIP), and the input embeddings of every modern LLM — descends from this thread.

## References

- [[efficient-estimation-of-word-representations-mikolov-2013]]
- [[distributed-representations-of-words-and-phrases-mikolov-2013]]
