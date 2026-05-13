---
title: Scaled Dot-Product Attention
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
status: complete
importance: high
---

# Scaled Dot-Product Attention

The specific [[attention-mechanism|attention function]] used in the [[transformer]]:

```
Attention(Q, K, V) = softmax(QKᵀ / √dₖ) V
```

where `Q`, `K`, `V` are query, key, and value matrices and `dₖ` is the key dimension. Proposed by [[noam-shazeer]].

## Why the √dₖ scaling

Without scaling, dot-product attention performs worse than additive attention for large `dₖ`. The intuition: if each component of `q` and `k` is mean-0 variance-1, the dot product `q · k = Σᵢ qᵢ kᵢ` has variance `dₖ`, so its magnitude grows as `√dₖ`. Large dot products push softmax into saturation regions with vanishing gradients. Dividing by `√dₖ` keeps the pre-softmax logits in a useful regime regardless of dimension.

## Why dot-product over additive

Additive attention (Bahdanau et al. 2014) computes compatibility via a small MLP. Dot-product attention is mathematically simpler but, more importantly, **faster** and more memory-efficient — it can be expressed as a single matrix multiplication, leveraging highly optimized BLAS / GPU matmul kernels. With the `√dₖ` scaling fix, dot-product matches or beats additive at all dimensions.

## Use

Scaled dot-product attention is the inner operation of [[multi-head-attention]], which is the canonical attention layer in every [[transformer-architecture|Transformer-based]] model.

## References

- [[attention-is-all-you-need-vaswani-2017]]
