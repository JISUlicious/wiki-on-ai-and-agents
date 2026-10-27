---
title: "Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality (Mamba-2)"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - mamba-2-transformers-are-ssms-dao-2024.md
tags:
  - 2024
  - paper
status: complete
importance: high
authors:
  - [[tri-dao]]
  - [[albert-gu]]
introduces:
  - [[mamba]]
year: 2024
venue: ICML 2024 (per external records; not stated in the text)
arxiv_id: "2405.21060"
---

# Mamba-2 / State Space Duality (SSD)

[arXiv:2405.21060](https://arxiv.org/abs/2405.21060), Dao and Gu, May 2024. The title is an homage to "Transformers are RNNs".

## The duality

A selective SSM `h_t = A_t h_{t−1} + B_t x_t`, `y_t = C_tᵀ h_t` is exactly **multiplication by a semiseparable matrix**: `y = M x`, with `M_ji = C_jᵀ A_j⋯A_{i+1} B_i`. An SSM with state size N corresponds to an N-semiseparable matrix.

Restrict A to a **scalar times identity** (A_t = a_t·I):

```
M = L ∘ (C Bᵀ),   L_ij = a_i × … × a_{j+1}  (i ≥ j)
⇒  Y = (L ∘ Q Kᵀ) · V      with C↔Q, B↔K, X↔V
```

- **What this is:** [[linear-attention]] with a data-dependent decay mask in place of softmax. The mask acts as a learned relative position encoding.
- **Why it matters:** SSMs and linear attention become **one family, with two algorithms**:
  - a linear-time recurrence for inference;
  - a matmul-heavy quadratic form for training.
- **What it doesn't cover:** SSD doesn't generalize *softmax* attention, which has no finite feature map.

## Mamba-2

- **Chunked SSD algorithm:** quadratic, attention-like matmuls inside chunks, plus a short recurrent scan across chunk states. It is **2–8× faster than Mamba's scan** and faster than FlashAttention-2 from 2K tokens (6× at 16K).
- **Bigger states:** the state is N = 64–256 instead of 16, at little slowdown. Associative recall improves steadily with N.
- **Transformer-like structure:**
  - head dimension 64–128;
  - parallel projections of A, X, B and C, which enable tensor parallelism;
  - extra normalization before the output projection.
- **Head pattern:** "multi-value" (B and C shared across heads) beat multi-head and multi-query variants.

## Results

- **Scaling** (125M–1.3B on the Pile): Mamba-2 Pareto-dominates Mamba and Transformer++.
- **At 2.7B on 300B tokens:** Pile perplexity 6.09, average 60.2. Mamba-2.8B scores 59.9.
- **Hybrids win:** **about 10% attention layers works best.**
  - At 350M: perplexity 8.60 with no attention, 8.26 with 6 of 48 layers attention, against 8.68 for Transformer++.
  - At 2.7B: **Mamba-2-Attention (58 SSD + 6 attention layers) reaches 5.95 / 61.0**, beating both pure Mamba-2 and Transformer++.
  - The authors' reading: attention acts "as a retrieval mechanism", so the state need not compress everything.

## Limitations

- Finite state makes associative recall and phonebook-style lookup hard.
- The copying and in-context-learning trade-offs are "not yet understood".
- A scalar A is slightly less expressive than a diagonal one.
- At short lengths (about 2K) the full model can train slower than a Transformer.

## Related

[[mamba]] · [[state-space-model]] · [[linear-attention]] · [[gated-delta-networks-yang-2024]] · [[jamba-hybrid-transformer-mamba-lieber-2024]] · [[nemotron-3-super-nvidia-2026]]
