---
title: Hyper-Connections (HC) and Manifold-Constrained Hyper-Connections (mHC)
type: concept
created: 2026-10-20
updated: 2026-10-20
sources:
  - depth-memory-gonzalez-martinez-2026.md
status: draft
importance: medium
builds-on:
  - [[residual-connection]]
---

# Hyper-Connections (HC) and mHC

**Hyper-Connections** (Zhu et al., 2025) replace the single [[residual-stream]] with an **expanded residual stream**: several parallel copies, in a matrix `X`. Each layer reads a learned combination of them, and learned maps mix them and write the layer output back in.

**Manifold-Constrained Hyper-Connections (mHC)** (Xie et al., 2026, [[deepseek]]) add constraints on these maps so that training stays stable at scale. Plain HC "becomes less stable as scale increases", as the DepthBench results cited in [[depth-memory-gonzalez-martinez-2026]] put it.

## Mechanics

The Depth Memory notation ([[depth-memory-gonzalez-martinez-2026]]) uses three maps generated from the current expanded state:
- **H_pre** builds the layer input as a query over the streams: `X q`.
- **H_res** mixes the streams.
- **H_post** writes the output back: `X ← X·H_res + v kᵀ`.

## Where it stands

- **Production.** The DeepSeek-V4 line uses mHC. DeepSeek-V4.1-Flash adds **Single-Pass mHC**, which reuses the previous block's input-mixing coefficients to halve activation traffic ([[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]]).
- **Taxonomy.** In the Depth Memory taxonomy, mHC is an *implicit* [[depth-mixing|depth mixer]]: a fixed-size state written additively, not appended.
- **Controlled comparisons** (Depth Memory):
  - mHC beats PreLN at 400M and 1.5B;
  - it roughly ties Block [[attention-residuals|AttnRes]] at 1.5B (validation perplexity 9.25 vs 9.26);
  - it trails [[linear-depth-memory|LDM]] (9.15), including when LDM's memory is cut to the same number of entries (400M).
  - Plain HC won only the deepest 400M shape in the aspect-ratio sweep. At 1.6B it stopped improving with depth.

## References

- [[depth-memory-gonzalez-martinez-2026]] · [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]]
- Primary papers not yet ingested: Zhu et al. 2025 (HC), Xie et al. 2026 (mHC).
