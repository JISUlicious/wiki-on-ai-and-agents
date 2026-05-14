---
title: LSTM (Long Short-Term Memory)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - hochreiter-lstm-1997.md
builds-on: "[[recurrent-neural-network]]"
status: complete
importance: high
---

# LSTM (Long Short-Term Memory)

LSTM is a recurrent neural network architecture introduced by [[long-short-term-memory-hochreiter-1997|Hochreiter & Schmidhuber (1997)]] to solve the [[vanishing-gradient-problem|vanishing-gradient problem]] in standard RNNs. The architecture uses a **memory cell** with a self-loop of weight 1.0 (a "constant error carousel" or CEC) and learned **multiplicative gates** that control when information enters, leaves, or is forgotten.

For roughly two decades (~1997–2017), the LSTM was the dominant architecture for sequence modeling — speech recognition, neural machine translation ([[seq2seq]]), language modeling, handwriting generation. The [[transformer-architecture|Transformer]] (2017) replaced it for most large-scale tasks by eliminating recurrence in favor of [[self-attention]], but LSTM-derived gating patterns persist in modern architectures (GRU, highway networks, Mamba's selective state-space layers).

## Architecture

A single LSTM cell maintains a hidden state `h_t` and a cell state `c_t`. At each time step, three (or four, in modern variants) gates compute scalar values in [0, 1]:

- **Input gate** `i_t`: controls how much of the candidate update writes to the cell.
- **Output gate** `o_t`: controls how much of the cell contents the hidden state exposes.
- **Forget gate** `f_t`: controls how much of the previous cell state is retained. (Added by Gers, Schmidhuber & Cummins, 2000 — the original 1997 paper had only input and output gates.)

The cell update is `c_t = f_t ⊙ c_{t-1} + i_t ⊙ c̃_t`, and the hidden state is `h_t = o_t ⊙ tanh(c_t)`. The crucial property: as long as the forget gate is open, gradient flows through `c_t` along a path with derivative 1, so the gradient neither vanishes nor explodes over long sequences.

## Why it mattered

- **Long-range credit assignment becomes tractable.** Vanilla RNNs trained with BPTT can effectively bridge ~10–20 time steps; LSTM bridges 1000+.
- **Empirical success**: powered ML breakthroughs in speech, translation, and language modeling from the late 2000s through the early 2010s — until the Transformer rendered it obsolete for the largest-scale tasks.
- **Architectural template**: every modern "gate" — Highway Networks, GRU, Residual gates, Mamba's selective state-space mechanism — is conceptually descended from LSTM gating.

## Why it lost ground to Transformers

- **Sequential computation**: LSTM hidden states must be computed serially along the time axis (`h_t` depends on `h_{t-1}`). Transformers eliminate this with [[self-attention]] over the full sequence, enabling massive GPU/TPU parallelism.
- **Limited effective context** in practice: while LSTMs *can* bridge 1000+ steps, in real-world large-scale training they tend to focus on much shorter spans.
- **Scaling**: Transformers scale more predictably and cleanly with parameters and data ([[scaling-laws]]).

## References

- [[long-short-term-memory-hochreiter-1997]]
