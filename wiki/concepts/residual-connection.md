---
title: Residual Connection
type: concept
created: 2026-05-13
updated: 2026-10-20
sources:
  - transformer-vaswani-2017.md
status: draft
importance: medium
---

# Residual Connection

A residual (or skip) connection adds the input of a sublayer to its output: `y = x + f(x)`. Introduced by He et al. (2016, ResNet) to enable training of very deep networks by giving gradients an identity path to flow through during backpropagation.

In the [[transformer]], every sublayer (attention or feed-forward) is wrapped as `LayerNorm(x + Sublayer(x))` in the original "post-norm" form. Many modern Transformer variants use "pre-norm" — `x + Sublayer(LayerNorm(x))` — for improved training stability at scale.

## Beyond the fixed sum: depth mixing

Since 2025 the unit-weight sum has had learned replacements, collectively called [[depth-mixing]]:
- [[hyper-connections|mHC]] (DeepSeek-V4);
- [[attention-residuals|Attention Residuals]] (Kimi K3);
- Gated Residual ([[qwen3.8-flash-next]]);
- [[linear-depth-memory|Linear Depth Memory]] (research).

The Depth Memory framework treats the plain residual as the one-stream special case of a memory with Read and Write operators ([[depth-memory-gonzalez-martinez-2026]]). Its ablations show the plain residual path is still worth keeping alongside the learned memory.

## References

- [[attention-is-all-you-need-vaswani-2017]]

## Lineage sources

- [[depth-memory-gonzalez-martinez-2026]] — residual connection as the simplest depth-memory stream; learned alternatives compared
