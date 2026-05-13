---
title: Attention Mechanism
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
status: complete
importance: high
---

# Attention Mechanism

An attention mechanism maps a *query* and a set of *key-value* pairs to an output, where the output is a weighted sum of the values and the weights are computed by a compatibility function between the query and the corresponding keys. Originally introduced in neural machine translation (Bahdanau et al. 2014) as an alignment mechanism between source and target sequences, attention has since become the central computational primitive of modern deep learning.

## Forms

- **Additive attention** (Bahdanau): compatibility via a small feed-forward network with one hidden layer.
- **Dot-product attention**: compatibility is just `qᵀk`.
- **[[scaled-dot-product-attention|Scaled dot-product attention]]** (Vaswani et al.): `qᵀk / √dₖ`, then softmax. This is the form used in the [[transformer]].
- **[[multi-head-attention|Multi-head attention]]**: parallel attention in multiple projected subspaces, concatenated.

## Variants by where Q/K/V come from

- **[[self-attention]]** (intra-attention): Q, K, V all come from the same sequence. Used to build contextualized representations.
- **Cross-attention**: Q from one sequence, K & V from another. Used in encoder-decoder Transformers and in retrieval-augmented models like [[rag]].
- **Masked / causal attention**: positions can only attend to earlier positions. Used in decoder-only language models ([[gpt-1]], [[gpt-2]], ...).

## Significance

The [[transformer]] showed that attention alone — without recurrence or convolution — is sufficient for state-of-the-art sequence modeling. This is what the paper title "Attention Is All You Need" refers to. Every modern LLM is built primarily out of attention layers.

## References

- [[attention-is-all-you-need-vaswani-2017]]
