---
title: "Long Short-Term Memory"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - hochreiter-lstm-1997.md
arxiv_id: null
venue: Neural Computation 9(8):1735–1780
authors:
  - "[[sepp-hochreiter]]"
  - "[[jurgen-schmidhuber]]"
first_author: "[[sepp-hochreiter]]"
introduces:
  - "[[lstm]]"
  - "[[vanishing-gradient-problem]]"
year: 1997
tags:
  - 1997
status: complete
importance: high
---

# Long Short-Term Memory

**Source**: `sources/hochreiter-lstm-1997.md` (extracted from `hochreiter-lstm-1997.pdf`)
**Venue**: Neural Computation 9(8):1735–1780, 1997
**Authors**: [[sepp-hochreiter]] (TU München), [[jurgen-schmidhuber]] (IDSIA, Lugano)

## Summary

Introduces the [[lstm|LSTM]] (Long Short-Term Memory) recurrent network architecture as a solution to the **[[vanishing-gradient-problem|vanishing/exploding gradient problem]]** in standard RNNs trained with Backpropagation Through Time (BPTT) or Real-Time Recurrent Learning (RTRL). Hochreiter had analyzed the problem in his 1991 diploma thesis; this paper presents the architectural fix.

The core idea is a **memory cell** with a **constant error carousel** (CEC) — a recurrent self-connection with weight 1.0 — that propagates error signals unchanged through time. Multiplicative **input** and **output gates** learn when to write to and read from the cell, protecting its contents from unrelated activity. Gradient flow through the CEC neither explodes nor vanishes (as long as truncation is applied at certain architecture-specific points), enabling the network to bridge time lags **in excess of 1000 discrete time steps** — far beyond what BPTT/RTRL, Elman nets, recurrent cascade-correlation, or any prior recurrent architecture could handle.

The LSTM is the conceptual ancestor of every later "context window" mechanism. It dominated sequence modeling for two decades (translation, speech, language modeling) until the [[transformer]] (2017) replaced recurrence entirely with [[self-attention]]. The fingerprints remain: gated mechanisms are used throughout modern architectures (GRU, highway networks, Mamba's selective state-space gates).

## Key Points

- **Problem named and analyzed**: error signals flowing back in time through standard RNNs either blow up or vanish exponentially — directly proportional to the spectral radius of the recurrent weight matrix. This is what makes long-range credit assignment intractable.
- **Constant Error Carousel (CEC)**: a memory cell with a fixed self-loop weight of 1.0, allowing error to flow through the cell unchanged.
- **Gating**: multiplicative input and output gates control when information enters and leaves the cell. (The forget gate that became standard in modern LSTMs was added later — Gers, Schmidhuber & Cummins, 2000.)
- **Truncated gradient flow**: gradients are truncated at certain points within the gate updates, which the paper shows does not impair the long-range learning the architecture is designed for.
- **Empirical**: LSTM solves long-time-lag tasks (≥1000 step delays, noisy inputs, distributed representations) that no prior RNN method had managed.
- **Computational cost**: O(1) per time step per weight — practical, not just theoretical.

## Entities Mentioned

- [[sepp-hochreiter]] — first author, then at TU München
- [[jurgen-schmidhuber]] — co-author, then at IDSIA (Lugano, Switzerland)

## Concepts Discussed

- [[lstm]] — the architecture introduced
- [[vanishing-gradient-problem]] — the problem solved
- [[recurrent-neural-network]] — the broader class LSTM belongs to

## Notable Quotes

> "Truncating the gradient where this does not do harm, LSTM can learn to bridge minimal time lags in excess of 1000 discrete time steps by enforcing constant error flow through 'constant error carousels' within special units." — Abstract

> "Multiplicative gate units learn to open and close access to the constant error flow." — Abstract

## References

_Original source: `sources/hochreiter-lstm-1997.md`_
_Canonical Neural Computation page is paywalled; this PDF is from the JKU mirror (https://www.bioinf.jku.at/publications/older/2604.pdf)._
