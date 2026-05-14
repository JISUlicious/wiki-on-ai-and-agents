---
title: Additive Attention
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - bahdanau-attention-2014.md
status: complete
importance: medium
---

# Additive Attention

Additive attention (also "Bahdanau attention") is the original soft-attention mechanism, introduced in [[neural-machine-translation-bahdanau-2014|Bahdanau, Cho, & Bengio (2014)]]. Given a query `q` (typically a decoder state) and a set of keys `(k_1, ..., k_n)` (typically encoder hidden states), the score for each key is computed via a small **MLP**:

```
e_j = v^T tanh(W_q q + W_k k_j)
```

Scores are softmax-normalized into attention weights `α_j` and used to form a context vector `c = Σ_j α_j v_j` over the corresponding values.

## Vs. scaled dot-product attention

The [[transformer]] (Vaswani et al. 2017) replaced additive attention with [[scaled-dot-product-attention|scaled dot-product attention]]: `score = q · k / √d_k`. The two perform comparably on most tasks, but dot-product is dramatically faster — it's expressible as a single matrix multiplication, exploiting highly optimized BLAS/GPU matmul kernels. Additive attention required per-pair MLP evaluation, which was slow.

The Transformer kept the structural idea (query, key, value, softmax, weighted sum) but swapped the scoring function. Today "attention" in the LLM literature almost always means scaled dot-product attention; additive attention is the historical original.

## Historical role

Bahdanau attention solved the **fixed-dim bottleneck** of [[seq2seq|seq2seq]] encoder-decoder models — decoders could now soft-search over the whole input rather than condition on a single compressed vector. This was the breakthrough that made neural machine translation competitive with phrase-based SMT and motivated the whole "attention is all you need" trajectory three years later.

## References

- [[neural-machine-translation-bahdanau-2014]]
