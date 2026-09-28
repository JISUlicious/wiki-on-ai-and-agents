---
title: Trained Sparse Attention
type: concept
created: 2026-09-10
updated: 2026-09-29
sources:
  - native-sparse-attention-yuan-2025.md
  - moba-mixture-of-block-attention-lu-2025.md
  - deepseek-v3.2-sparse-attention-deepseek-2025.md
  - flashmemory-lookahead-sparse-attention-wang-2026.md
  - misa-mixture-of-indexer-sparse-attention-zhou-2026.md
  - gated-sparse-attention-shen-2026.md
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
| **Gated Sparse Attention** (Jan 2026) | DSA-style indexer with sigmoid scores plus Qiu et al.'s element-wise G1/G2 gates on values and head outputs; adaptive k in [256, 4,096] | token | **from scratch**: 1.7B, 400B SlimPajama tokens, 4K window, YaRN to 128K | RULER-128K **62.2 vs 31.7** dense — but *gated-only dense* scores 58.8, so the gain is mostly gating's; "11×" prefill at 128K is relative and unscoped |
| **MISA** (May 2026) | route each query to **8 of the indexer's 64 heads** with a parameter-free router over pooled keys; main attention unchanged | token, head-axis cut | **training-free retrofit — but only onto models already trained with DSA** (V3.2, GLM-5) | **3.82× on the indexer kernel** (H200); LongBench 50.85 vs DSA 51.05; kernel-only, and the indexer is ~3% of per-token compute at 128K |

[[misa-mixture-of-indexer-sparse-attention-zhou-2026|MISA]] adds a **fourth position** the from-scratch / adapt / retrofit axis did not have: a training-free retrofit that presupposes the trained column. It cannot sparsify a dense model; it makes an already-sparse model's selector cheaper. [[gated-sparse-attention-shen-2026|GSA]] is the cautionary entry — its own ablation shows the sparse-only baseline is *already more stable* than dense (8.7 vs 12.3 loss spikes per 100K steps), so the paper does not show sparsity causing instability, and its long-context win belongs mostly to a generic gating trick.

> [!success] Resolved contradiction — how many heads does DSA's lightning indexer have?
> [[gated-sparse-attention-shen-2026|GSA]] §2.2 says "typically 4" and builds its theoretical cost figures on H^I = 4. The released DeepSeek-V3.2 and V3.2-Exp configs say `index_n_heads: 64`, which is also what [[misa-mixture-of-indexer-sparse-attention-zhou-2026|MISA]] uses and what its 8-of-64 routing presupposes. The config is authoritative; GSA's theoretical numbers are off by that factor.
>
> **Resolved 2026-09-29:** checked against the released `config.json` for both DeepSeek-V3.2 and V3.2-Exp on [[huggingface|Hugging Face]]: `index_n_heads: 64`, `index_head_dim: 128`, `index_topk: 2048`. The config is authoritative; GSA's "typically 4" is wrong.

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

- [[native-sparse-attention-yuan-2025]] · [[moba-mixture-of-block-attention-lu-2025]] · [[deepseek-v3.2-sparse-attention-deepseek-2025]] · [[gated-sparse-attention-shen-2026]] · [[misa-mixture-of-indexer-sparse-attention-zhou-2026]] · [[flashmemory-lookahead-sparse-attention-wang-2026]] · [[minference-dynamic-sparse-attention-jiang-2024]]
