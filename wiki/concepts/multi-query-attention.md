---
title: Multi-Query Attention (MQA)
type: concept
created: 2026-10-23
updated: 2026-10-23
sources:
  - fast-transformer-decoding-one-write-head-shazeer-2019.md
  - deepseek-v2-deepseek-ai-2024.md
  - cross-layer-attention-kv-cache-brandon-2024.md
status: draft
importance: high
builds-on:
  - [[multi-head-attention]]
---

# Multi-Query Attention (MQA)

**MQA** keeps many query heads but gives all of them **a single shared key head and value head**. It was introduced by [[noam-shazeer]] in 2019 ([[fast-transformer-decoding-one-write-head-shazeer-2019]]).

## Cache size

The [[kv-cache]] per token per layer is `2·d_h`, against `2·n_h·d_h` for [[multi-head-attention|MHA]]: an **h-fold** reduction.

## Why it was invented

During decoding, MHA reloads all of K and V for every new token. The paper's memory-to-arithmetic ratio for incremental MHA is `Θ(n/d + 1/b)`, which approaches 1 when sequences are long or the batch is small, so decoding becomes bandwidth-bound. MQA cuts the `n/d` term by `h`. The paper measured about 12× faster greedy decoding (46 → 3.8 µs per token) with training cost unchanged.

## Descendants

- **[[grouped-query-attention|GQA]]** interpolates with G KV heads: GQA-1 is MQA and GQA-H is MHA. It recovers most of MHA's quality and has become the default.
- **[[cross-layer-kv-sharing|CLA]]** stacks sharing across layers on top of MQA, the combination where its gains were "most robust".
- **[[multi-head-latent-attention|MLA]]** reaches a cache equal to GQA with about 2.25 groups, while claiming better quality than MHA.
- **Already-compressed caches resist further compression.** Falcon-7B uses MQA, so KIVI needed 4 bits rather than 2 for it ([[kivi-2bit-kv-cache-quantization-liu-2024]]).

> [!warning] Contradiction: how much quality MQA costs
> - **Shazeer 2019** (encoder-decoder translation and a ~200M LM): "only minor quality degradation": −0.2 dev BLEU, +0.3 perplexity, and *higher* beam-4 test BLEU.
> - **DeepSeek-V2's Table 8** (7B dense decoders, 1.33T tokens): MMLU **37.9 vs 45.2** (MHA) and C-Eval 30.0 vs 42.9 ([[deepseek-v2-deepseek-ai-2024]]).
> - **GQA paper** (T5-XXL): MQA −1.27% ([[key-value-compress-kv-cache-compression-survey-2025]]).
>
> The cost grows on knowledge-heavy benchmarks at larger scale; it isn't a fixed property of MQA.

## References

- [[fast-transformer-decoding-one-write-head-shazeer-2019]] · [[deepseek-v2-deepseek-ai-2024]] · [[grouped-query-attention-ainslie-2023]] · [[cross-layer-attention-kv-cache-brandon-2024]] · [[kv-cache-size-vs-active-parameters]]
