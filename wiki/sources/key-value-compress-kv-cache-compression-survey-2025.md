---
title: "Key, Value, Compress: A Systematic Exploration of KV Cache Compression Techniques"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - key-value-compress-kv-cache-compression-survey-2025.md
tags:
  - 2025
  - survey
status: complete
importance: low
authors:
  - Neusha Javidnia
  - Bita Darvish Rouhani
  - Farinaz Koushanfar
year: 2025
venue: IEEE CICC 2025
arxiv_id: "2503.11816"
---

# Key, Value, Compress (KV-cache compression survey)

[arXiv:2503.11816](https://arxiv.org/abs/2503.11816), IEEE CICC 2025: Javidnia (UCSD), Darvish Rouhani (NVIDIA), Koushanfar (UCSD). A short survey of about 7 pages.

## Taxonomy: by which dimension is compressed

| Dimension | Families | Examples |
|---|---|---|
| **Layers** | cross-layer sharing; dropping attention layers | [[you-only-cache-once-yoco-sun-2024\|YOCO]]; Attention Drop (half of LLaMA-3-70B's attention layers pruned at comparable quality) |
| **Heads** | sharing or compressing across heads | [[multi-query-attention\|MQA]], [[grouped-query-attention\|GQA]], [[multi-head-latent-attention\|MLA]] |
| **Tokens** | no cache (SSMs); attention-based pruning and eviction; prompt compression; learned merging | Mamba, Mamba2-Hybrid; LazyLLM, FastGen, [[h2o-heavy-hitter-oracle-zhang-2023\|H2O]]; LLMLingua, CPC; DMC |
| **Hidden dimension** | quantization (mentioned only briefly) | — |

[[streamingllm-attention-sinks-xiao-2023|StreamingLLM]], [[snapkv-li-2024|SnapKV]] and [[kivi-2bit-kv-cache-quantization-liu-2024|KIVI]] are **not covered**.

**Motivating number:** a 65B GQA model with an 8-bit cache needs about 86 GB for 512K tokens, which is more than one H100.

## Numbers collected (from the original papers; "not directly comparable")

- **Accuracy change against each method's own baseline:**
  - YOCO-3B +2.42%; Mamba2-Hybrid-8B +4.98%;
  - GQA-8-XXL −0.21%; MQA-XXL −1.27%;
  - DMC 2× +1.63%;
  - **H2O −5.11%**, apparently taken from the DMC paper's comparison;
  - CPC +13.64% and LongLLMLingua +9.77% (prompt compression).
- **Speedups:**
  - YOCO prefill 2.87× at 32K; LazyLLM 2.03× time-to-first-token;
  - H2O 1.86×; FastGen 1.56×; DMC 8× gives 1.25×.

## Conclusions

- **Training from scratch** gives better accuracy at high cost. **Post-training** methods are stable. **Training-free** prompt compression can even improve accuracy.
- **Input-heavy tasks** benefit more than generation-heavy ones.
- **Call for better evaluation:** same baselines, data, batch and hardware. The authors flag a "significant gap" in measuring GPU memory and throughput.

## Related

[[kv-cache]] · [[kv-cache-eviction]] · [[kv-cache-quantization]] · [[cross-layer-kv-sharing]] · [[kv-cache-size-vs-active-parameters]]
