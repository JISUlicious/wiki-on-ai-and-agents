---
title: xLSTM
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - xlstm-extended-lstm-beck-2024.md
tags:
  - model
status: draft
importance: medium
family: lstm
builds-on:
  - [[lstm]]
---

# xLSTM

The 2024 revival of the [[lstm|LSTM]] by [[sepp-hochreiter]]'s group (JKU Linz and NXAI). It has two cell types:
- **sLSTM:** exponential gating and memory mixing, which enable state tracking but can't be parallelized.
- **mLSTM:** a parallelizable d×d matrix memory with a covariance update, effectively gated [[linear-attention]].

**Results** (1.3B, 300B tokens):
- validation perplexity 8.89, against Mamba's 9.14 and Llama's 9.44;
- trained at 2K, it extrapolates to 16K at perplexity about 9, where Llama reaches 338.

Paper: [[xlstm-extended-lstm-beck-2024]].

## Lineage

[[long-short-term-memory-hochreiter-1997|LSTM (1997)]] → **xLSTM (2024)**. The ablation climbs from vanilla LSTM (perplexity 2418) through a residual backbone, up-projection and exponential gating to matrix memory (13.48).

## References

- [[xlstm-extended-lstm-beck-2024]] · [[lstm]] · [[state-space-model]]
