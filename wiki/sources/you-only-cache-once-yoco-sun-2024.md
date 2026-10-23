---
title: "You Only Cache Once: Decoder-Decoder Architectures for Language Models (YOCO)"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - you-only-cache-once-yoco-sun-2024.md
tags:
  - 2024
  - paper
status: complete
importance: medium
authors:
  - Yutao Sun
  - Li Dong
  - Yi Zhu
  - Shaohan Huang
  - Wenhui Wang
  - Shuming Ma
  - Quanlu Zhang
  - Jianyong Wang
  - Furu Wei
introduces:
  - [[cross-layer-kv-sharing]]
year: 2024
venue: arXiv preprint
arxiv_id: "2405.05254"
---

# You Only Cache Once (YOCO)

[arXiv:2405.05254](https://arxiv.org/abs/2405.05254)

| | |
|---|---|
| **Authors** | Sun, Dong (equal contribution), Zhu, Huang, Wang, Ma, Zhang, Wang, Wei |
| **Affiliations** | [[microsoft-research]], Tsinghua University |
| **Code** | aka.ms/YOCO |

## Summary

YOCO is a **decoder-decoder** design.
- **Self-decoder (bottom L/2 layers):** uses constant-memory attention, either **gated retention** (the default; a data-dependent decay) or [[sliding-window-attention]] with window C = 1,024. It produces **one global KV cache**.
- **Cross-decoder (top L/2 layers):** every layer **reuses that single cache** through causal cross-attention, each with its own query projection, compatible with GQA.

The result is the most extreme case of [[cross-layer-kv-sharing]]: one cache for the whole stack.

## Memory and prefill

| | KV memory | Prefill |
|---|---|---|
| Transformer | O(L·N·D) | O(L·N²·D) |
| YOCO | **O((N + L)·D)** ("roughly saves L times GPU memory") | **O(L·N·D)**, with an **early exit**: nothing in the cross-decoder is needed to build the cache, so prefill stops at layer L/2 |

The "once" in the name ignores the self-decoder's small constant-size cache.

## Results

- **YOCO-3B** (26 layers, hidden 3072, 24 Q / 8 KV heads, 1.6T tokens): eval-harness average 0.636, against 0.619 for OpenLLaMA-3B-v2. Across 160M–13B scaling runs it is "comparable" to a Llama-style Transformer, with gated retention better than sliding-window.
- **Long context:**
  - 1M context with near-perfect single-needle retrieval.
  - Multi-needle at 128K: 0.98 / 0.98 / 0.84 / 0.56 for 1 / 2 / 4 / 8 needles.
- **Memory** (H100):
  - 12.4 GB in total at 1M tokens; the Transformer needs **9.4×** more.
  - The KV cache per token is **24× smaller at 1.2B and 80× smaller at 65B**.
  - At 65B, 1 GB holds 128K tokens, against 1.6K for a GQA Transformer.
- **Prefill and throughput:**
  - 512K-token prefill takes 180 s → under 6 s (**30.3×**); the speedup is 71.8× at 1M.
  - Throughput at 512K is 4.5 → 43.1 tok/s (9.6×).
  - The baseline Transformer already used GQA, Flash-Decoding and kernel fusion.

## Limitations

- **Pretraining required:** the design must be trained from scratch.
- **Profiling scale:** profiling covers 3B models.
- **Fixed-state cost:** the self-decoder's constant state is excluded from the "once" accounting.

## Related

[[cross-layer-kv-sharing]] · [[kv-cache]] · [[sliding-window-attention]] · [[linear-attention]] · [[cross-layer-attention-kv-cache-brandon-2024]] · [[hysparse2-hybrid-sparse-attention-two-level-kv-sharing-wei-2026]] (YOCO-style KV bridging) · [[kv-cache-size-vs-active-parameters]]
