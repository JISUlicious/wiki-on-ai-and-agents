---
title: "Jamba: A Hybrid Transformer-Mamba Language Model"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - jamba-hybrid-transformer-mamba-lieber-2024.md
tags:
  - 2024
  - paper
status: complete
importance: medium
authors:
  - Opher Lieber
  - Barak Lenz
  - Hofit Bata
  - Gal Cohen
  - Yonatan Belinkov
  - Shai Shalev-Shwartz
  - Yoav Shoham
introduces:
  - [[jamba]]
year: 2024
venue: arXiv preprint
arxiv_id: "2403.19887"
---

# Jamba: A Hybrid Transformer-Mamba Language Model

[arXiv:2403.19887](https://arxiv.org/abs/2403.19887) (v2, 2024-07-03), from [[ai21-labs]]: Lieber, Lenz (equal contribution), Bata, Cohen and 20 others. Apache-2.0.

## Summary

[[jamba|Jamba]] interleaves attention layers with Mamba ([[state-space-model|SSM]]) layers and adds [[mixture-of-experts|MoE]]. The paper calls it the "first production-grade Attention-SSM hybrid model".

**Configuration:**
- 4 blocks × 8 layers = 32 layers, with **attention : Mamba = 1 : 7**, so only 4 attention layers;
- MoE every 2nd layer, 16 experts, top-2;
- **52B total / 12B active**; fits on one 80 GB GPU;
- GQA in the attention layers, no positional embeddings;
- trained to 1M context; the released model supports 256K.

## KV cache at 256K (16-bit, the paper's Table 1)

| Model | Total params | Active params | KV cache |
|---|---|---|---|
| Llama-2 | 6.7B | 6.7B | 128 GB |
| Mistral | 7.2B | 7.2B | 32 GB |
| Mixtral | 46.7B | 12.9B | 32 GB |
| **Jamba** | **52B** | **12B** | **4 GB** |

- **The 8× is pure layer ratio.** Only 1 in 8 layers caches K/V, so against Mixtral (already GQA) the saving is exactly 8×.
- **Mamba state isn't counted.** The fixed-size Mamba state is excluded from "KV cache".
- **Active parameters don't set the cache.** Mixtral and Jamba have similar active parameters but caches 8× apart. This is the point made in [[kv-cache-size-vs-active-parameters]].

## Results

- **Capacity and speed.**
  - On one 80 GB A100 it fits "2x the context length of Mixtral and 7x that of Llama-2-70B", about 140K tokens.
  - Throughput is about 3× Mixtral's at large batch and at 128K. The comparisons were unoptimized.
- **Quality** (Jamba vs Mixtral):

  | MMLU | HellaSwag | GSM8K | HumanEval | BBH |
  |---|---|---|---|---|
  | 67.4 vs 70.6 | 87.1 vs 86.7 | 59.9 vs 60.4 | 29.3 vs 34.8 | 45.4 vs 50.3 |

- **Ablations.**
  - 1:3 and 1:7 ratios score alike. The hybrid beats pure attention and pure Mamba.
  - **Pure Mamba fails at in-context format-following** (IMDB 48.8, against 84.1 for attention and 90.9 for the hybrid). The authors find induction-like heads in the attention layers.
  - MoE adds +1.5 on their aggregate score.

## Limitations

- **Base model only.** It has no alignment and "should not be used in production".
- **Training stability.** Mamba needed RMSNorm to avoid loss spikes.
- **Quality gaps.** It trails Mixtral on MMLU, HumanEval and BBH.

## Related

[[jamba]] · [[ai21-labs]] · [[state-space-model]] · [[linear-attention]] · [[mixture-of-experts]] · [[kv-cache]] · [[kimi-linear-kimi-team-2025]] (a later 3:1 hybrid) · [[kv-cache-size-vs-active-parameters]]
