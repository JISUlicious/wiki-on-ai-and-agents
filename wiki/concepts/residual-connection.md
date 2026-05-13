---
title: Residual Connection
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
status: draft
importance: medium
---

# Residual Connection

A residual (or skip) connection adds the input of a sublayer to its output: `y = x + f(x)`. Introduced by He et al. (2016, ResNet) to enable training of very deep networks by giving gradients an identity path to flow through during backpropagation.

In the [[transformer]], every sublayer (attention or feed-forward) is wrapped as `LayerNorm(x + Sublayer(x))` in the original "post-norm" form. Many modern Transformer variants use "pre-norm" — `x + Sublayer(LayerNorm(x))` — for improved training stability at scale.

## References

- [[attention-is-all-you-need-vaswani-2017]]
