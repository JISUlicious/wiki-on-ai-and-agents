---
title: Vanishing-Gradient Problem
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - hochreiter-lstm-1997.md
status: draft
importance: medium
---

# Vanishing-Gradient Problem

In a deep or recurrent neural network, gradients computed by backpropagation can shrink (or explode) exponentially in depth or sequence length. **Vanishing** gradients prevent the network from learning long-range dependencies; **exploding** gradients cause unstable, oscillating weights. The problem was analyzed in detail in Sepp Hochreiter's 1991 diploma thesis and is the foundational obstacle the [[lstm|LSTM]] was designed to address.

For a recurrent network, the gradient of the loss at time T with respect to the recurrent weights at time t involves a product of T−t Jacobian matrices. If the largest singular value of these Jacobians is consistently < 1, the gradient decays exponentially toward 0; if > 1, it grows without bound. In practice, both regimes are common with standard activations and initialization.

## Mitigations

- **[[lstm|LSTM]]** (1997) and GRU (2014): gated memory cells with a constant-error-carousel path that preserves gradient flow.
- **[[residual-connection|Residual connections]]** (ResNet, 2015): direct identity paths for gradients to flow through, enabling networks of 100+ layers.
- **[[layer-normalization]]** and batch normalization: keep activations in well-conditioned regimes.
- **Careful initialization** (Xavier, He init).
- **Gradient clipping**: a simple guard against exploding gradients in RNNs.
- **[[self-attention|Self-attention]] / [[transformer-architecture|Transformers]]** (2017): eliminate the recurrence path entirely, so credit assignment doesn't need to flow through O(T) sequential operations.

## References

- [[long-short-term-memory-hochreiter-1997]]
