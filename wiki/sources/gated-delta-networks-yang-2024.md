---
title: "Gated Delta Networks: Improving Mamba2 with Delta Rule"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - gated-delta-networks-yang-2024.md
tags:
  - 2024
  - paper
status: complete
importance: high
authors:
  - Songlin Yang
  - Jan Kautz
  - Ali Hatamizadeh
introduces:
  - [[gated-deltanet]]
year: 2024
venue: ICLR 2025
arxiv_id: "2412.06464"
---

# Gated Delta Networks (Gated DeltaNet)

[arXiv:2412.06464](https://arxiv.org/abs/2412.06464), **ICLR 2025**. Authors: Yang ([[mit|MIT]]; an [[nvidia]] intern), Kautz and Hatamizadeh (NVIDIA). Code: NVlabs/GatedDeltaNet.

## Two complementary memory rules

| Rule | Update | Good at | Bad at |
|---|---|---|---|
| Linear attention | `S_t = S_{t−1} + v_t k_tᵀ` | — | "memory collisions" beyond about d stored pairs |
| Gating (Mamba-2) | `S_t = α_t S_{t−1} + v_t k_tᵀ` | **rapid erasure** | forgets every association uniformly |
| Delta rule (DeltaNet) | `S_t = S_{t−1}(I − β_t k_t k_tᵀ) + β_t v_t k_tᵀ` | **targeted overwrite** of one key | can't clear memory quickly |
| **Gated delta rule** | **`S_t = S_{t−1}(α_t(I − β_t k_t k_tᵀ)) + β_t v_t k_tᵀ`** | both | — |

- **Limits of the gate:** α_t → 0 clears the state; α_t → 1 gives back the pure delta rule.
- **As test-time learning:** each step is **one step of online SGD** on `½‖S k_t − v_t‖²`, with learning rate β_t and adaptive weight decay α_t. That links it to TTT, [[titans]] and [[test-time-memory]].
- **Training:** chunkwise and parallel, via the WY/UT transform, so it runs on tensor cores. Throughput is about DeltaNet's and slightly below Mamba-2.
- **Block:** Llama-style. q, k and v pass through a short convolution and SiLU, q and k are L2-normalized, and there's an output gate. All of these matter in ablations.

## Results (1.3B parameters, 100B FineWeb-Edu tokens)

| Model | Wiki ppl | Avg acc | Retrieval avg | LongBench |
|---|---|---|---|---|
| **Gated DeltaNet** | **16.42** | **55.32** | 30.6 | **16.6** |
| Mamba-2 | 16.56 | 54.89 | 29.8 | 13.5 |
| DeltaNet | 17.71 | 52.14 | 26.2 | 13.6 |
| Transformer++ | 18.53 | 52.25 | **37.0** | 11.0 |
| H1 (Gated DeltaNet + sliding-window attention) | 16.07 | **56.40** | 39.0 | 17.8 |
| H2 (Mamba-2 + Gated DeltaNet + sliding-window attention) | **15.91** | 56.18 | **40.1** | **18.4** |

**S-NIAH** shows the complementarity directly:
- Gated DeltaNet scores 91.8 / 92.2 / 84.2 on the three variants, against Mamba-2's 30.4 / 56.2 / 47.6 and DeltaNet's 98.8 / 18.6 / 47.0.
- Mamba-2's decay hurts retention; gating helps filtering; the delta rule helps memorization.

## Where it went

The paper names no production adopters. In this wiki, Gated DeltaNet is the layer in **36 of 48 layers of [[qwen3.8-flash-next|Qwen3.8-Flash-Next]]**, and the base of Kimi's channel-wise **KDA** ([[kimi-linear-kimi-team-2025]]). The FPGA accelerator in [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]] targets it. These adoptions come from those sources, not this paper.

## Limitations

- **Pure recurrent models still trail Transformers on real-world retrieval** (30.6 vs 37.0); hybrids close the gap.
- Mixed length extrapolation.
- Small models aren't instruction-tuned.

## Related

[[gated-deltanet]] · [[linear-attention]] · [[state-space-model]] · [[mamba-2-transformers-are-ssms-dao-2024]] · [[rwkv-7-goose-peng-2025]] · [[a-hippocampus-for-linear-attention-cui-2026]]
