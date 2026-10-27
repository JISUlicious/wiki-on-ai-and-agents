---
title: Mamba (and Mamba-2)
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - mamba-selective-state-spaces-gu-2023.md
  - mamba-2-transformers-are-ssms-dao-2024.md
tags:
  - model
status: draft
importance: high
family: mamba
builds-on:
  - [[state-space-model]]
release_date: 2023-12-01
---

# Mamba (and Mamba-2)

The selective [[state-space-model]] architecture by [[albert-gu]] and [[tri-dao]].
- **Mamba** (December 2023) made the SSM's Δ, B and C depend on the input, and computed it with a hardware-aware scan ([[mamba-selective-state-spaces-gu-2023]]).
- **Mamba-2** (May 2024) restricted A to a scalar times identity. That makes the layer a form of masked [[linear-attention]] (**state space duality**) and lets it train 2–8× faster with 4–16× larger states ([[mamba-2-transformers-are-ssms-dao-2024]]).

## Lineage

- **[[s4-structured-state-spaces-gu-2021|S4]] (2021):** time-invariant, HiPPO initialization, trained as a convolution.
- **Mamba (2023):** selective (input-dependent) and scan-based. It was the first attention-free model to match Transformer++ scaling; Mamba-3B matches Transformers twice its size, with 5× inference throughput.
- **Mamba-2 (2024):** SSD with a scalar A, chunked matmuls and larger N. Its own ablation finds **about 10% attention layers** works best.
- **[[gated-deltanet|Gated DeltaNet]] (2024):** adds the delta rule to Mamba-2's gate. It is the line production hybrids adopted (Qwen3.8-Flash-Next, Kimi's KDA).

**Mamba-based production hybrids:** [[jamba]] (AI21, attention:Mamba 1:7) and [[nemotron-3-super-nvidia-2026|Nemotron 3]] (NVIDIA, a hybrid Mamba-Transformer MoE). Distillation from attention models into Mamba: [[attention-to-mamba-cross-architecture-distillation-moudgil-2026]].

## References

- [[mamba-selective-state-spaces-gu-2023]] · [[mamba-2-transformers-are-ssms-dao-2024]] · [[state-space-model]]
