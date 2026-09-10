---
title: Trained Sparse Attention
type: concept
created: 2026-09-10
updated: 2026-09-10
sources:
  - native-sparse-attention-yuan-2025.md
  - moba-mixture-of-block-attention-lu-2025.md
  - deepseek-v3.2-sparse-attention-deepseek-2025.md
  - flashmemory-lookahead-sparse-attention-wang-2026.md
status: complete
importance: high
---

# Trained Sparse Attention

Sparse attention where the selector is **in the training loop** — the model learns, or is adapted, to attend only to what a cheap selector picks — as opposed to the training-free retrofits on [[sparse-attention]] and [[sparse-prefill]] (MInference, FlexPrefill, XAttention, SALE, TriangleMix) that estimate importance from a frozen model's own attention and must therefore be conservative.

The shared design across the lineage: a cheap per-query selector (pooled keys, compressed tokens, or a learned indexer) chooses a subset of keys; full attention runs only over that subset; and because the model trained with the selector, it can tolerate far more aggressive sparsity than a retrofit can. The 2025–2026 frontier open models — DeepSeek-V3.2, Kimi, [[qwen3.8-flash-next|Qwen3.8-Flash-Next]] — all ship some form of it.

## The three papers

| | Selector | Granularity | Training | Headline |
|---|---|---|---|---|
| **NSA** (DeepSeek, Feb 2025; ACL 2025 best paper) | three parallel branches — compressed coarse tokens, selected fine-grained blocks, sliding window — mixed by a learned gate; blocks aligned to GQA groups | block | **from scratch**: 27B total / 3B active, 260–270B tokens | attention kernel **9.0× forward / 6.0× backward at 64K**, 11.6× decode; matches or beats full attention on general, long-context and reasoning benchmarks |
| **MoBA** (Moonshot, Feb 2025) | per-query top-k over mean-pooled key blocks, MoE-style gate; the current block always attended | block | **adapt**: fine-tune a dense model, can switch full↔sparse | **6.5× at 1M**, 16× at 10M, ~1× at 512K; deployed for Kimi's long-context serving |
| **DSA** (DeepSeek-V3.2, Dec 2025) | lightning indexer — few heads, FP8, `ReLU(q·k)` scores — then top-2,048 entries per query under MLA | **token** | **adapt**: 2.1B-token dense warm-up training only the indexer, then 943.7B-token sparse stage on the whole model | parity with V3.1-Terminus on short and long tasks; O(L²) → O(L·k) main attention |

Two details from the source pages that the headlines hide:

- **Every NSA speedup is a kernel number.** There is no end-to-end prefill, TTFT or throughput figure anywhere in the paper. The wiki's attention-share rule on [[sparse-prefill]] applies: attention is ~⅓ of prefill at 32K and ~70% at 128K on a fully-quadratic model, so a 9× kernel is a much smaller wall-clock number, and on a hybrid with few quadratic layers, smaller still.
- **MoBA is still quadratic.** With fixed block count *n* and top-*k*, its cost is O(N²·k/n + N·n); at k/n = 3/64 that is a constant-factor reduction, which is exactly why it is ~1× at 512K and only pays past 1M.

## Retrofit versus trained: the actual trade

| | Training-free retrofit | Trained |
|---|---|---|
| Cost to adopt | none — works on any checkpoint | NSA: a pretraining run; MoBA: a fine-tune; DSA: ~1T tokens of continued pre-training |
| Sparsity it can afford | conservative — MInference is **0.5× at 32K** and 1.9–3.4× at 128K on the kernel | aggressive — DSA keeps 2,048 of 128K keys per query |
| Where it pays | ≥64K on fully-quadratic models; ≤1.11× on hybrids at any length | designed into the model, so from the length the vendor chose |
| Quality | near-lossless with visible task-level losses (PassageRetrieval, multi-doc QA) | parity claimed at the model level; task-level breakdowns rarely published |

The practical consequence for anyone choosing a model rather than a kernel: **a 2026 model that ships trained sparse attention has already taken the long-context prefill win**, and stacking a training-free retrofit on top of it has no published evidence behind it.

## Where the lineage goes next

[[flashmemory-lookahead-sparse-attention-wang-2026|DeepSeek-V4's lookahead sparse attention]] extends DSA's indexer toward ultra-long context. The hybrid designs — [[qwen3.8-flash-next]] at 3:1 GDN-to-sparse-attention, Kimi's KDA — combine [[linear-attention]] layers with a minority of trained sparse-attention layers, which is what makes token-axis methods, not attention-axis ones, the right retrofit for them (see UniPrefill on [[sparse-prefill]]).

## Related

- [[sparse-attention]] · [[sparse-prefill]] · [[flash-attention]] · [[grouped-query-attention]] · [[multi-head-latent-attention]]
- [[linear-attention]] · [[qwen3.8-flash-next]] · [[kimi-k3-moonshot-2026]] · [[long-context-llm]]
- [[deepseek]] · [[moonshot-ai]] · [[arithmetic-intensity]]

## References

- [[native-sparse-attention-yuan-2025]] · [[moba-mixture-of-block-attention-lu-2025]] · [[deepseek-v3.2-sparse-attention-deepseek-2025]] · [[flashmemory-lookahead-sparse-attention-wang-2026]] · [[minference-dynamic-sparse-attention-jiang-2024]]
