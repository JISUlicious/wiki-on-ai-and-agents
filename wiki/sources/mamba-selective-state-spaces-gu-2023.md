---
title: "Mamba: Linear-Time Sequence Modeling with Selective State Spaces"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - mamba-selective-state-spaces-gu-2023.md
tags:
  - 2023
  - paper
status: complete
importance: high
authors:
  - [[albert-gu]]
  - [[tri-dao]]
introduces:
  - [[mamba]]
year: 2023
venue: arXiv preprint (later COLM 2024, per external records)
arxiv_id: "2312.00752"
---

# Mamba: Linear-Time Sequence Modeling with Selective State Spaces

[arXiv:2312.00752](https://arxiv.org/abs/2312.00752), Gu ([[cmu|CMU]]) and Dao ([[princeton-university|Princeton]]), December 2023. Code: state-spaces/mamba.

## Framing

"Sequence modeling is compressing context into a state."
- **Attention** "explicitly does not compress context at all", which is why it needs a [[kv-cache]].
- **Recurrent models** are efficient because their state is finite, but they are only as good as that compression.
- **Prior SSMs** were all linear time-invariant ([[s4-structured-state-spaces-gu-2021|S4]]), so they couldn't *select* what to keep.

## Selection ("S6")

Make the step size Δ and the projections B and C **functions of the input**. A stays fixed and diagonal, with N = 16 per channel by default.

```
B_t = Linear_N(x_t),  C_t = Linear_N(x_t),  Δ_t = softplus(param + Linear(x_t))
Ā_t = exp(Δ_t A),  B̄_t = (Δ_t A)⁻¹(exp(Δ_t A) − I)·Δ_t B_t      (zero-order hold)
h_t = Ā_t h_{t−1} + B̄_t x_t,   y_t = C_t h_t
```

- **Large Δ** resets the state and focuses on the current token; **small Δ** keeps the state and ignores the token.
- **With N = 1** this reduces exactly to classic RNN gating.
- **Δ is the most important selective parameter.** Raising N from 1 to 16 only helps when B and C are selective.

## Making it fast

Selection breaks the convolution trick, so Mamba computes the recurrence with a **hardware-aware parallel scan**:
- **Kernel fusion:** Δ, A, B and C are loaded into SRAM, discretized and scanned there.
- **Never materialized:** the (B, L, D, N) state never touches GPU main memory (HBM).
- **Recomputation:** intermediate states are recomputed in the backward pass.

Memory matches a FlashAttention Transformer. The scan beats FlashAttention-2 beyond 2K tokens.

**The Mamba block** merges an H3 block with a gated MLP (expansion 2, short convolution, SiLU) and is stacked homogeneously, with no attention and no MLP blocks.

## Results

- **Synthetic tasks:**
  - Selective copying: 99.8% (S4 without a gate: 18.3%).
  - Induction heads trained at length 256 **extrapolate perfectly to about 1M tokens**.
- **Language** (Pile, 300B tokens): the first attention-free model to match a Llama-style Transformer++ in scaling.
  - Mamba-2.8B averages 63.3, against Pythia-2.8B 59.1, Pythia-6.9B 61.7 and RWKV-7.4B 62.5.
  - The claim: it "matches Transformers twice its size".
- **Inference:** "**5× higher throughput** than Transformers". With no KV cache, much larger batches fit.
- **DNA and audio:** perplexity improves with context up to 1M (DNA). On SC09 speech generation, FID is 0.94 vs 1.99 for SaShiMi.

## Limitations (stated)

- **"No free lunch":** selection can hurt continuous signals like audio, where time-invariant models excel.
- **Scale:** evaluated only below 7B parameters.
- **Ecosystem:** it's unknown whether Transformer tooling (fine-tuning, RLHF, quantization) transfers.
- **Recall:** the fixed-state recall weakness is raised later, in [[mamba-2-transformers-are-ssms-dao-2024|Mamba-2]] and [[gated-delta-networks-yang-2024|Gated DeltaNet]].

## Related

[[mamba]] · [[state-space-model]] · [[s4-structured-state-spaces-gu-2021]] · [[mamba-2-transformers-are-ssms-dao-2024]] · [[jamba-hybrid-transformer-mamba-lieber-2024]] · [[flash-attention]]
