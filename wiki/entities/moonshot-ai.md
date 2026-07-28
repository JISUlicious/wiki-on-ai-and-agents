---
title: Moonshot AI
type: entity
created: 2026-07-28
updated: 2026-07-28
sources:
  - flash-kda-moonshot-2026.md
  - attention-residuals-kimi-team-2026.md
status: draft
importance: medium
tags:
  - org
---

# Moonshot AI

**Moonshot AI** is a Chinese AI lab, publishing as the **Kimi Team**, behind the **Kimi** model family. Within this wiki its distinguishing characteristic is a consistent focus on **attention-architecture efficiency** — it contributes both novel mechanisms and the production kernels that make them fast, rather than treating the two separately.

## Contributions in this wiki

| Work | What it is |
|---|---|
| [[kimi-linear-kimi-team-2025\|Kimi Linear]] | introduces **Kimi Delta Attention (KDA)**, a [[linear-attention]] mechanism refining Gated DeltaNet with finer-grained gating |
| [[flash-kda-moonshot-2026\|FlashKDA]] | the CUTLASS kernel implementation of KDA — ~2× over the `flash-linear-attention` baseline on H20, MIT licensed |
| [[attention-residuals-kimi-team-2026\|Attention Residuals (AttnRes)]] | replaces fixed unit-weight residual accumulation with softmax attention over preceding layer outputs |

The pattern worth noting: **KDA and AttnRes are separate research lines that converge in the flagship model.** Per [vLLM's engineering blog](https://vllm.ai/blog/2026-07-27-k3), **Kimi K3** is built on both — a Mixture-of-Experts model activating **16 of 896 experts** per token via "Stable LatentMoE" (quantile-balanced latent-space experts), with a **1M-token context window** and native vision. KDA is described there as *"a hybrid recurrent + full-attention stack"*, confirming Kimi Linear's hybrid rather than pure-linear design.

> [!note] Draft
> Created during the FlashKDA ingest. Founding date, funding, headcount and the earlier Kimi model lineage (K1/K1.5/K2) are not yet recorded — this page currently covers only the attention-architecture line documented in this wiki.

## Related

- [[linear-attention]] · [[mixture-of-experts]] · [[flash-attention]] (the analogous kernel effort for exact attention) · [[vllm]] (day-0 K3 serving support)

## References

- [[flash-kda-moonshot-2026]] · [[kimi-linear-kimi-team-2025]] · [[attention-residuals-kimi-team-2026]]
- [vLLM: Kimi K3 day-0 support](https://vllm.ai/blog/2026-07-27-k3) — secondary source for K3 architecture details.
