---
title: Multi-Head Attention
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
status: complete
importance: high
---

# Multi-Head Attention

Multi-head attention runs `h` parallel [[scaled-dot-product-attention|scaled dot-product attention]] operations in different learned projection subspaces, then concatenates and re-projects the results. Introduced in "[[attention-is-all-you-need-vaswani-2017|Attention Is All You Need]]" (proposed by [[noam-shazeer]]), it is the standard form of attention in [[transformer-architecture|Transformer-based]] models.

## Definition

Given queries `Q`, keys `K`, values `V` of dimension `dmodel`:

```
MultiHead(Q, K, V) = Concat(head₁, ..., head_h) Wᴼ
where head_i = Attention(Q Wᵢ_Q, K Wᵢ_K, V Wᵢ_V)
```

Each `Wᵢ_Q ∈ ℝ^(dmodel × dₖ)`, `Wᵢ_K ∈ ℝ^(dmodel × dₖ)`, `Wᵢ_V ∈ ℝ^(dmodel × dᵥ)`, and `Wᴼ ∈ ℝ^(h·dᵥ × dmodel)`. Typical settings: `h=8`, `dₖ = dᵥ = dmodel/h = 64`.

## Why multiple heads

A single attention head averages information across attended positions, which can blur distinct relationships. Multiple heads let the model jointly attend to information from different representation subspaces and at different positions — e.g., one head tracks syntactic agreement, another tracks long-range coreference. Vaswani et al.'s ablations (their Table 3 row A) show single-head attention is ~0.9 BLEU worse than h=8, with too many heads also degrading quality.

## Computational cost

Crucially, the total FLOPs per layer are *the same* as a single attention head with full `dmodel` dimension, because each head operates on `dmodel/h`-dimensional vectors. Multi-head is essentially free.

## Where it's used

- Encoder self-attention (e.g., [[bert]])
- Causal decoder self-attention (GPT family)
- Cross-attention (decoder attending to encoder output, as in the original [[transformer]] for translation, or in [[rag|RAG]] for retrieval-augmented generation)

## References

- [[attention-is-all-you-need-vaswani-2017]]
