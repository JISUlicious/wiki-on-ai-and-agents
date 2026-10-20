---
title: Attention Residuals (AttnRes)
type: concept
created: 2026-10-20
updated: 2026-10-20
sources:
  - attention-residuals-kimi-team-2026.pdf
  - depth-memory-gonzalez-martinez-2026.md
status: draft
importance: medium
builds-on:
  - [[residual-connection]]
---

# Attention Residuals (AttnRes)

**AttnRes** replaces the fixed unit-weight sum of a pre-norm [[residual-connection]] with **softmax attention over earlier layer outputs**. It was introduced by the Kimi Team at [[moonshot-ai]] ([[attention-residuals-kimi-team-2026]], arXiv:2603.15031).

## The problem it fixes

In pre-norm transformers every layer's output is added with weight 1. The hidden state grows with depth, and each layer's contribution gets diluted ("PreNorm dilution").

## How it works

Each layer has a learned query, a per-layer pseudo-query. Its input is a softmax-weighted mix of the embedding and all earlier layer outputs, with RMS-normed keys. Each layer therefore picks which earlier representations to use.

| Variant | Attends over | Cost |
|---|---|---|
| **Full AttnRes** | every earlier layer output | memory grows with depth, O(Ld) |
| **Block AttnRes** | the embedding, completed block sums and the current block's partial sum (about 8 blocks) | O(Nd); near drop-in |

## Where it stands

- **Production.** [[kimi-k3-moonshot-2026|Kimi K3]] uses Block AttnRes with 8 blocks of 12 layers, alongside KDA [[linear-attention]].
- **In the Depth Memory taxonomy** ([[depth-memory-gonzalez-martinez-2026]]), AttnRes is an *explicit* [[depth-mixing|depth mixer]]: a key–value cache over layers that is read by attention and written by appending.
- **Benchmarks.** In that paper's controlled comparisons:
  - Block AttnRes had the **best 400M zero-shot average** (50.61);
  - [[linear-depth-memory|LDM]] had lower perplexity at both 400M and 1.5B;
  - LDM's throughput exceeded Full AttnRes everywhere and roughly matched Block AttnRes.

## Related

- [[hyper-connections]] — the implicit (fixed-size, expanded-stream) alternative
- [[residual-stream]] · [[qwen3.8-flash-next]] (Gated Residual, a related learned residual)

## References

- [[attention-residuals-kimi-team-2026]] · [[depth-memory-gonzalez-martinez-2026]] · [[kimi-k3-moonshot-2026]]
