---
title: Cross-Layer KV Sharing
type: concept
created: 2026-10-23
updated: 2026-10-23
sources:
  - cross-layer-attention-kv-cache-brandon-2024.md
  - you-only-cache-once-yoco-sun-2024.md
status: draft
importance: medium
builds-on:
  - [[kv-cache]]
  - [[multi-query-attention]]
---

# Cross-Layer KV Sharing

[[multi-query-attention|MQA]] and [[grouped-query-attention|GQA]] share K/V **across heads**. Cross-layer sharing goes one step further and shares them **across layers**: some layers compute K/V and others reuse them. In the cache formula `c_tok = Σ_layers 2·n_kv·d_h`, it removes layers from the sum ([[kv-cache-size-vs-active-parameters]]).

## The spectrum

| Method | Sharing | Cache reduction | Evidence |
|---|---|---|---|
| [[cross-layer-attention-kv-cache-brandon-2024\|CLA2]] | pairs of adjacent layers | 2× | 1B/3B from scratch; < 1% perplexity change on top of MQA |
| CLA3, CLA4 | 3 or 4 layers | 3–4× | still Pareto-beats MQA, but worse than CLA2 |
| [[you-only-cache-once-yoco-sun-2024\|YOCO]] | **one global cache** reused by every upper layer; the lower half uses constant-memory attention | about L× (24× at 1.2B, 80× at 65B) | 3B on 1.6T tokens matches baselines; 1M context; 30× faster prefill at 512K via early exit |

## Notes

- **CLA buys capacity, not per-step speed.** Each layer still reads the shared cache, so per-step attention bandwidth is unchanged. YOCO's prefill gain is different: it comes from skipping the cross-decoder entirely when building the cache.
- **Pretraining required.** Both require pretraining from scratch, unlike [[kv-cache-quantization]] or [[kv-cache-eviction]].
- **2026 descendants:** HySparse2's "YOCO-style KV bridging" ([[hysparse2-hybrid-sparse-attention-two-level-kv-sharing-wei-2026]]) and DeepSeek-V4.1's cross-layer sparse KV reuse ([[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]]). [[claa-cross-layer-attention-aggregation-mcdanel-2026|CLAA]] is a different method despite the similar name.

## Related

[[kv-cache]] · [[multi-query-attention]] · [[multi-head-latent-attention]] · [[linear-attention]] · [[key-value-compress-kv-cache-compression-survey-2025]]
