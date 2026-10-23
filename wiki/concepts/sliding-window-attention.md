---
title: Sliding-Window Attention (SWA)
type: concept
created: 2026-10-23
updated: 2026-10-23
sources:
  - mistral-7b-jiang-2023.md
  - you-only-cache-once-yoco-sun-2024.md
  - streamingllm-attention-sinks-xiao-2023.md
status: draft
importance: medium
builds-on:
  - [[multi-head-attention]]
---

# Sliding-Window Attention (SWA)

Each token attends only to the previous **W** positions. The [[kv-cache]] per layer is therefore capped at W tokens, held in a **rolling buffer**: position i goes to slot i mod W. That makes cache memory `O(W)` instead of `O(T)`.

## Where it appears

- **[[mistral-7b|Mistral 7B]]:** W = 4096 over 32 layers, so information can reach about 131K tokens back through depth. The paper claims 8× less cache at 32K "without impacting the model quality", but shows no ablation for it ([[mistral-7b-jiang-2023]]).
- **[[you-only-cache-once-yoco-sun-2024|YOCO]]:** one of two self-decoder options (window 1,024). Gated retention was better.
- **Shazeer's local attention:** a window of 32, combined with [[multi-query-attention|MQA]] ([[fast-transformer-decoding-one-write-head-shazeer-2019]]).

## Caveat: plain window attention breaks without sinks

Evicting the first tokens destroys perplexity: Llama-2-7B goes from 9.59 to 3,359.95 with a 4K window. The cause is [[attention-sink|attention sinks]]. A trained-in window, as in Mistral, is a different setup from applying a window at inference time to a model trained on full attention. In the second case, keep the sinks ([[streamingllm-attention-sinks-xiao-2023]]).

## Related

[[kv-cache-eviction]] · [[sparse-attention]] · [[linear-attention]] · [[kv-cache-size-vs-active-parameters]]
