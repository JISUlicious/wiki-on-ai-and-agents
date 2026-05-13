---
title: Layer Normalization
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
status: draft
importance: medium
---

# Layer Normalization

Layer normalization (Ba et al. 2016) normalizes activations across the feature dimension (rather than the batch dimension as in batch norm), making it suitable for variable-length sequence models and small or unstable batches. It is the standard normalization layer in [[transformer-architecture|Transformer-based]] models.

In the original [[transformer]] it is applied as `LayerNorm(x + Sublayer(x))` (post-norm) around every attention and feed-forward sublayer. Modern Transformer variants often use pre-norm (`x + Sublayer(LayerNorm(x))`) for training stability, and some replace LayerNorm with RMSNorm (Zhang & Sennrich 2019) for efficiency.

## References

- [[attention-is-all-you-need-vaswani-2017]]
