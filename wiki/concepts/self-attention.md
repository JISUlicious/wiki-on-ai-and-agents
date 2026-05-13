---
title: Self-Attention
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
status: complete
importance: high
---

# Self-Attention

Self-attention (also called *intra-attention*) is an [[attention-mechanism]] that relates different positions of a single sequence to one another in order to compute a representation of that sequence. Each position emits a query, key, and value; the output at position `i` is a weighted sum of all positions' values, weighted by the compatibility between position `i`'s query and every other position's key.

Self-attention is the defining computational primitive of the [[transformer]]. Unlike recurrence (which computes hidden states sequentially) or convolution (which computes local neighborhoods), self-attention relates any two positions in **constant** sequential complexity. This is what makes Transformers parallelizable and effective at long-range dependencies.

## Concrete form

In the original Transformer, self-attention is implemented as [[scaled-dot-product-attention]] over packed matrices:

```
Attention(Q, K, V) = softmax(QKᵀ / √dₖ) V
```

In practice, [[multi-head-attention|multi-head self-attention]] is used: Q, K, V are projected into `h` parallel subspaces, attention is computed in each, and outputs are concatenated.

## Causal / masked variant

Decoder-only models like [[gpt-1]], [[gpt-2]], [[gpt-3]], and [[gpt-4]] use **masked** (causal) self-attention: position `i` may only attend to positions `≤ i`. This enables autoregressive generation by ensuring the prediction at position `i` doesn't depend on future tokens.

## Significance

Self-attention was used in NLP before the [[transformer]] paper (e.g., reading comprehension, summarization, entailment models), but the Transformer was the first transduction model to rely *entirely* on self-attention, eliminating recurrence and convolution. Subsequent models — [[bert]] (encoder self-attention), the GPT family (causal self-attention), [[vit]] (self-attention over image patches) — built on this pattern.

## References

- [[attention-is-all-you-need-vaswani-2017]]
