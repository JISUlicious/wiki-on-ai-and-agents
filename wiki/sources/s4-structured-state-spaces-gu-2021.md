---
title: "Efficiently Modeling Long Sequences with Structured State Spaces (S4)"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - s4-structured-state-spaces-gu-2021.md
tags:
  - 2021
  - paper
status: complete
importance: high
authors:
  - [[albert-gu]]
  - Karan Goel
  - Christopher Ré
first_author: [[albert-gu]]
introduces:
  - [[state-space-model]]
year: 2021
venue: ICLR 2022 (per external records; not stated in the text)
arxiv_id: "2111.00396"
---

# S4: Efficiently Modeling Long Sequences with Structured State Spaces

[arXiv:2111.00396](https://arxiv.org/abs/2111.00396), Gu, Goel and Ré ([[stanford-university|Stanford]]), 2021; code in HazyResearch/state-spaces. This is the paper that made [[state-space-model|SSMs]] a practical deep-learning layer.

## The model

A continuous linear system, `x'(t) = A x(t) + B u(t)`, `y = C x(t)`, discretized with step Δ (bilinear):

```
Ā = (I − Δ/2·A)⁻¹(I + Δ/2·A),   B̄ = (I − Δ/2·A)⁻¹ ΔB
recurrent view:   x_k = Ā x_{k−1} + B̄ u_k,   y_k = C x_k            (O(N) per step: fast generation)
convolution view: y = K̄ * u,  K̄ = (CB̄, CĀB̄, …, CĀ^{L−1}B̄)        (parallel training via FFT)
```

The parameters are the same at every step (**linear time-invariant**), so the same layer can train as a convolution and generate as an RNN.

## The two ideas

1. **HiPPO initialization** of A (`A_nk = −√(2n+1)√(2k+1)` for n > k, `−(n+1)` on the diagonal) lets the state *memorize the input history*. Swapping a random A for HiPPO takes sequential MNIST from 60% to 98%. Ablations show **the gains come mainly from HiPPO**, not from the parameterization.
2. **Normal Plus Low-Rank (NPLR/DPLR) parameterization**, `A = VΛV* − PQᵀ`. The naive diagonalization of HiPPO has entries up to about 2^(4N/3) and is numerically infeasible. NPLR reduces the convolution kernel to a **Cauchy kernel**, computable in Õ(N + L) via the Woodbury identity. That is **up to 30× faster with 400× less memory** than its predecessor LSSL.

## Results

- **Long Range Arena:** average **86.09** (updated) or 80.48 (original). It is the **first model to solve Path-X** (16,384 steps): 96.35, against chance for every prior model. The best prior average was 59.37.
- **Raw speech** (length 16,000): 98.32%, and 96.30% at half the sampling rate with no retraining.
- **Generation:** about **60× faster** than a Transformer through the recurrent state.
- **WikiText-103:** perplexity 20.95, against the Transformer's 20.51. That is close, but language modelling still trailed.

## Limitations

- A gap to Transformers on language modelling remains. LTI dynamics can't select content based on the input, which [[mamba-selective-state-spaces-gu-2023|Mamba]] later fixes.
- The fast Cauchy algorithm wasn't implemented on GPU; the paper used a naive O(NL) kernel.

## Related

[[state-space-model]] · [[mamba-selective-state-spaces-gu-2023]] · [[albert-gu]] · [[recurrent-neural-network]]
