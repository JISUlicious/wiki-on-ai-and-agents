---
title: "Reducing Transformer Key-Value Cache Size with Cross-Layer Attention (CLA)"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - cross-layer-attention-kv-cache-brandon-2024.md
tags:
  - 2024
  - paper
status: complete
importance: medium
authors:
  - William Brandon
  - Mayank Mishra
  - Aniruddha Nrusimha
  - Rameswar Panda
  - Jonathan Ragan-Kelley
introduces:
  - [[cross-layer-kv-sharing]]
year: 2024
venue: arXiv preprint (under review)
arxiv_id: "2405.12981"
---

# Cross-Layer Attention (CLA)

[arXiv:2405.12981](https://arxiv.org/abs/2405.12981)

| | |
|---|---|
| **Authors** | Brandon, Mishra (equal contribution), Nrusimha, Panda, Ragan-Kelley |
| **Affiliations** | MIT CSAIL, MIT-IBM Watson AI Lab |

## Summary

[[multi-query-attention|MQA]] and [[grouped-query-attention|GQA]] share K/V **across heads**. CLA also shares them **across adjacent layers**: only some layers compute KV projections, and the rest reuse an earlier layer's cache ([[cross-layer-kv-sharing]]).
- **CLA2** pairs adjacent layers and **halves the cache**; CLA3 shares across 3, and so on.
- It is orthogonal to MHA, GQA and MQA, and works best with MQA.

## Results

Llama-like models trained from scratch on SlimPajama: 1B on 30B tokens, 3B on 100B tokens.

| Configuration | Bytes / token | Val PPL |
|---|---|---|
| 1B H128-MQA | 10,240 | 12.39 |
| **1B H128-MQA-CLA2** | **5,120** | **12.43** |
| 1B H64-MQA (same memory as CLA2) | 5,120 | 12.74 |
| 3B H128-MQA | — | 9.52 |
| **3B MQA-CLA2 (d_head 128)** | half | **9.34** |

- **MQA + CLA2 beats baselines with the same memory** by 0.21–0.48 PPL in the 1B sweep. The authors' summary is 2× KV reduction with "less than 1% change" in perplexity.
- **CLA3 and CLA4** still Pareto-beat plain MQA, but are worse than CLA2.
- **Non-uniform sharing patterns** (keep the ends, dense front, dense back) are worse than uniform sharing.
- **GQA4 + CLA2 did not help;** only GQA2 + CLA2 matched MQA-CLA2.
- **Learning rate:** CLA models prefer a higher one.

## What it does and doesn't buy

- **Capacity, not speed.** The gain is cache capacity: bigger batches and longer context. **It has no direct effect on per-step attention bandwidth**, because each layer still reads the shared cache separately.
- **Parallelism constraint.** Under pipeline parallelism, layers that share a cache must sit in the same stage.
- **Not measured:** end-to-end inference efficiency is left to future work. Only 1B and 3B scales were tested.

## Related

[[cross-layer-kv-sharing]] · [[you-only-cache-once-yoco-sun-2024]] · [[multi-query-attention]] · [[grouped-query-attention]] · [[kv-cache]] · [[claa-cross-layer-attention-aggregation-mcdanel-2026]] (a different 2026 method with a similar name) · [[kv-cache-size-vs-active-parameters]]
