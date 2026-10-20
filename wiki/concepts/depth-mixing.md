---
title: Depth Mixing
type: concept
created: 2026-10-20
updated: 2026-10-20
sources:
  - depth-memory-gonzalez-martinez-2026.md
  - attention-residuals-kimi-team-2026.pdf
status: draft
importance: medium
builds-on:
  - [[residual-connection]]
---

# Depth Mixing

**Depth mixing** is how a network combines representations *across layers*, for each token. It is the third axis of transformer design ([[depth-memory-gonzalez-martinez-2026]]):
- **Sequence mixing** works across tokens: attention, [[linear-attention]], [[state-space-model|SSMs]].
- **Channel mixing** works across features: MLPs, [[mixture-of-experts|MoE]].
- **Depth mixing** works across layers.

The default since ResNet is the [[residual-connection]]: every layer's output is added, with weight 1, to a running sum (the [[residual-stream]]). From 2025 to 2026, several labs replaced that fixed sum with learned mechanisms, and some of these now ship in frontier models.

## The designs

| Design | What changes | Kind | Where it appears |
|---|---|---|---|
| PreLN residual | Fixed unit-weight sum | baseline | almost every LLM |
| [[hyper-connections\|Hyper-Connections (HC)]] / **mHC** | An *expanded* residual stream (several parallel streams), with learned maps to read, mix and write it. mHC constrains those maps for stability. | implicit, fixed size | DeepSeek-V4 line ([[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]]) |
| [[attention-residuals\|Attention Residuals (AttnRes)]] | Softmax attention over earlier layer (or block) outputs | explicit, growing (Full) or block summaries (Block) | [[kimi-k3-moonshot-2026\|Kimi K3]] (Block, 8 blocks) |
| Gated Residual | 4 branches with data-dependent read gates and scalar write gates | implicit | [[qwen3.8-flash-next]] |
| [[linear-depth-memory\|Linear Depth Memory (LDM)]] | Residual stream plus a matrix memory updated by exact recursive least squares | implicit, fixed size | research (400M–1.6B) |

## The memory view

The Depth Memory framework ([[depth-memory-gonzalez-martinez-2026]]) treats any depth mixer as streams with **Read** and **Write** operators. It separates two kinds:
- **Explicit** mixers keep a copy of every layer's output (Full AttnRes). They are expressive, but memory grows with depth.
- **Implicit** mixers compress outputs into a fixed-size state (mHC, LDM). This is the depth-axis version of the softmax-vs-linear-attention trade-off on the sequence axis.

Under this view, the **sequence-mixing toolbox carries over to depth**:
- linear attention becomes an outer-product memory;
- the delta rule and exact regression become depth-regression solvers.

Depth has one advantage: layers already run in order, so exact sequential solvers like RLS are affordable. On the token axis they would block parallel training.

## What the evidence says (2026)

- **All the learned mixers tested beat PreLN** on perplexity at 400M–1.6B (AttnRes, mHC, LDM).
- **The ranking among them is small and scale-dependent.** In Depth Memory, LDM < mHC ≈ Block AttnRes on perplexity. Among the learned mixers, zero-shot averages differ by at most about 1 point (0.3 points at 1.5B).
- **Keeping a plain residual path matters.** LDM without its separate residual stream is worse than PreLN.
- **Production adoption** is the strongest evidence of value: AttnRes in Kimi K3, mHC in DeepSeek-V4, Gated Residual in Qwen3.8-Flash-Next.
- **No controlled comparison at frontier scale exists in the wiki.**

## Related

- [[residual-stream]] — the interpretability framing of the default stream ("a shared memory every layer reads from and writes to"), which Depth Memory makes literal
- [[looped-transformer]] · [[recurrent-depth-transformer]] — a different use of depth: re-applying the same layers, not mixing across them
- [[test-time-memory]] — regression-style memories on the sequence axis

## References

- [[depth-memory-gonzalez-martinez-2026]] · [[attention-residuals-kimi-team-2026]] · [[kimi-k3-moonshot-2026]]
