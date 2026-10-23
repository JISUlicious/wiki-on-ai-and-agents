---
title: KV-Cache Eviction
type: concept
created: 2026-10-23
updated: 2026-10-23
sources:
  - streamingllm-attention-sinks-xiao-2023.md
  - h2o-heavy-hitter-oracle-zhang-2023.md
  - snapkv-li-2024.md
status: draft
importance: medium
builds-on:
  - [[kv-cache]]
---

# KV-Cache Eviction

Keeping only a **subset of tokens' K/V** under a fixed budget, which shrinks the `T` in `KV = c_tok·T·B·b_kv`. The tokens are still prefilled first. That is the difference from [[token-pruning]] and [[sparse-prefill]], which skip computing them at all.

## Three policies

| Method | What's kept | When | Headline |
|---|---|---|---|
| [[streamingllm-attention-sinks-xiao-2023\|StreamingLLM]] (ICLR 2024) | **4 [[attention-sink\|sink]] tokens + recent window** | continuously | stable perplexity to 4M tokens; 22.2× faster than sliding window with recomputation |
| [[h2o-heavy-hitter-oracle-zhang-2023\|H2O]] (NeurIPS 2023) | **heavy hitters** (highest accumulated attention) + recent tokens | every decode step | full quality at a 20% budget; up to 29× throughput |
| [[snapkv-li-2024\|SnapKV]] (2024) | prompt positions voted by an **observation window** at the prompt's end, plus that window | **once, at prefill**; generated tokens kept | 380K tokens on one A100; 3.6× faster, 8.2× less memory at 16K |

## The shared weakness: eviction is irreversible

- **H2O:** "evicted KV embeddings become inaccessible"; dropping critical ones "can cause a severe functional collapse".
- **StreamingLLM:** answers "drop to zero" once the evidence falls out of the cache, and it loses to plain truncation on LongBench. It "does not … enhance their long-term memory".
- **SnapKV:** different instructions over the same document attend to *different* positions. A cache compressed for one question may lack what the next turn needs.
- **Survey number:** H2O shows −5.11% on MMLU, CS-QA and HumanEval in a cross-paper comparison ([[key-value-compress-kv-cache-compression-survey-2025]]).

Two 2026 methods in the wiki revive dropped tokens for a later turn: LazyLLM and SlimInfer ([[sparse-prefill]]). Quantization ([[kv-cache-quantization]]) keeps every token at lower precision instead, which is safer for retrieval.

## Related

[[kv-cache]] · [[attention-sink]] · [[token-pruning]] · [[sparse-prefill]] · [[sparse-attention]] · [[sliding-window-attention]] · [[kv-cache-size-vs-active-parameters]]
