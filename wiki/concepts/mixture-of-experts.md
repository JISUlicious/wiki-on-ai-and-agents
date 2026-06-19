---
title: Mixture of Experts (MoE)
type: concept
created: 2026-06-19
updated: 2026-06-19
sources:
  - glm-5-2-zai-2026.md
  - nemotron-3-super-nvidia-2026.md
  - minimax-m2-series-minimax-2026.md
  - scalable-moe-training-megatron-core-yan-2026.md
  - mai-thinking-1-microsoft-2026.md
status: complete
importance: high
tags:
  - 2026
---

# Mixture of Experts (MoE)

A **Mixture of Experts (MoE)** is an architecture that replaces a dense feed-forward layer with **many parallel "expert" sub-networks plus a router (gating network)** that, for each token, activates only a small subset of experts. This decouples **total parameters** (capacity) from **active parameters** (per-token compute): a model can hold trillions of parameters while each token only pays for a few billion. MoE is the dominant way 2026 frontier models scale capacity without proportional inference cost — most large models in this wiki are MoE.

## How it works

- **Experts**: N independent FFN blocks per MoE layer.
- **Router / gating**: a learned function scores experts per token and selects the **top-k** (often k=1–2). Only selected experts run.
- **Sparsity**: "active params" = the experts actually fired per token; "total params" = all experts summed. E.g. [[glm-5-2|GLM-5.2]] is 744B total / 40B active; [[mai-thinking-1|MAI-Thinking-1]] is 1T total / 35B active.
- **Load balancing**: auxiliary losses (or loss-free schemes) spread tokens across experts so capacity isn't wasted on a few hot experts — a central training difficulty.

## Why it matters

MoE breaks the dense scaling law's tie between capability and serving cost. The trade-offs are systems-level: routing adds communication, experts must be sharded across devices, and memory holds *all* experts even though compute touches few — which is why MoE progress is as much a **training-infrastructure** story as an architecture one. See [[scalable-moe-training-megatron-core-yan-2026|Megatron-Core MoE training]] for the systems side.

## Examples in this wiki

- [[glm-5-2]] — 744B/40B-active, coding-first, 1M context.
- [[mai-thinking-1]] — 1T/35B-active, "hill-climbing machine."
- [[nemotron-3-super-nvidia-2026|Nemotron 3 Super]] — MoE hybrid Mamba-Transformer.
- [[minimax-m2-series-minimax-2026|MiniMax-M2]] — MoE LLM series.
- [[scalable-moe-training-megatron-core-yan-2026]] — scalable MoE training systems.
- [[attention-residuals-kimi-team-2026]], [[composer-2-technical-report-cursor-2026]] — further MoE-based systems.

## Related

- [[transformer-architecture]] — MoE swaps the dense FFN sublayer for routed experts.
- [[scaling-laws]] — MoE changes the compute/capacity frontier.
- State-space models / Mamba — often hybridized with MoE (e.g. Nemotron 3's hybrid Mamba-Transformer).

## References

- [[scalable-moe-training-megatron-core-yan-2026]] · [[glm-5-2-zai-2026]] · [[nemotron-3-super-nvidia-2026]] · [[minimax-m2-series-minimax-2026]] · [[mai-thinking-1-microsoft-2026]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]] for full grouping.

- [[attention-to-mamba-cross-architecture-distillation-moudgil-2026]] — Attention to Mamba A Recipe for Cross Architecture
- [[massive-activations-attention-sinks-sun-2026]] — The Spike, the Sparse and the Sink Anatomy
- [[minimax-sparse-attention-lai-2026]] — MiniMax Sparse Attention
