---
title: Recurrent Neural Network (RNN)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - hochreiter-lstm-1997.md
status: draft
importance: medium
---

# Recurrent Neural Network (RNN)

A recurrent neural network is a neural network with cyclic connections, allowing it to maintain a hidden state that propagates information across time steps. Given an input sequence `(x_1, ..., x_T)`, an RNN computes hidden states `h_t = f(W_h h_{t-1} + W_x x_t)`, where the same weights are applied at every step.

RNNs were the dominant architecture for sequence modeling from the 1980s through ~2017, when [[transformer-architecture|Transformers]] replaced them at scale. Training RNNs is hampered by the [[vanishing-gradient-problem|vanishing-gradient problem]], which makes naive RNNs ineffective at bridging long time lags. Gated variants — [[lstm|LSTM]] (1997) and GRU (2014) — addressed this and became the standard.

## Why Transformers replaced RNNs

- **Sequential computation**: `h_t` depends on `h_{t-1}`, so RNNs cannot parallelize along the time axis during training. Transformers process the full sequence in parallel via [[self-attention]].
- **Long-range modeling**: even LSTMs struggle past a few hundred steps in practice. Transformers attend to any position in O(1) sequential cost.
- **Scaling cleanliness**: see [[scaling-laws]].

RNNs persist where streaming / online inference matters (online ASR, time-series with infinite history), or in hybrid architectures (RWKV, Mamba) that combine recurrence with parallel training.

## References

- [[long-short-term-memory-hochreiter-1997]]
