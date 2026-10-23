---
title: "Fast Transformer Decoding: One Write-Head is All You Need"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - fast-transformer-decoding-one-write-head-shazeer-2019.md
tags:
  - 2019
  - paper
status: complete
importance: high
authors:
  - [[noam-shazeer]]
first_author: [[noam-shazeer]]
introduces:
  - [[multi-query-attention]]
year: 2019
venue: arXiv preprint
arxiv_id: "1911.02150"
---

# Fast Transformer Decoding: One Write-Head is All You Need

[arXiv:1911.02150](https://arxiv.org/abs/1911.02150), a single-author paper by [[noam-shazeer]] (Google), November 2019. It introduces **[[multi-query-attention|multi-query attention (MQA)]]**.

## The idea

Keep **many query heads** but give all of them **one shared key head and one shared value head**. Dropping the head axis from K and V shrinks the [[kv-cache]] h-fold: `[b, h, m, k] → [b, m, k]`.

## Why it helps: the bandwidth analysis

Assume n tokens, width d, h heads, batch b, and k = d/h. The ratio of memory access to arithmetic:

| Setting | Memory : arithmetic |
|---|---|
| Training-style parallel MHA | `O(1/k + 1/(bn))`, which is low |
| Incremental (decode) MHA | `Θ(n/d + 1/b)` |
| Incremental MQA | `Θ(1/d + n/(dh) + 1/b)` |

- **Decode MHA is bandwidth-bound:** "When n ≈ d or b ≈ 1, the ratio is close to 1, causing memory bandwidth to be a major performance bottleneck."
- **MQA fixes the n/d term:** "We have reduced the offensive n/d by a factor of h." Larger batches fix the 1/b term.

This is the same decode-not-prefill argument that [[grouped-query-attention-ainslie-2023|GQA]] later inherits.

## Results

- **Translation (WMT14 EN-DE):** 6-layer encoder-decoder, h = 8, 211M parameters. The MQA model widens d_ff to keep parameters equal.

  | Attention | ln(PPL) dev | BLEU dev | BLEU test (beam 1 / 4) |
  |---|---|---|---|
  | multi-head | 1.424 | 26.7 | 27.7 / 28.4 |
  | **multi-query** | 1.439 | 26.5 | 27.5 / **28.5** |
  | multi-head, fewer heads or smaller dims (h = 1 … 8 variants) | 1.480–1.518 | 25.8–26.2 | — |

- **Decode cost** (TPUv2, µs per output token, sequence 128):

  | Attention | Greedy decoder | Beam-4 decoder |
  |---|---|---|
  | multi-head | 46 | 203 |
  | multi-query | **3.8** | **32** |

  About **12×** faster greedy and **6×** faster beam-4 decoding. Training cost is unchanged (13.2 → 13.0).
- **Language modelling (Billion-Word):** dev perplexity 29.9 (MHA) vs 30.2 (MQA).
- **Verdict:** MQA is "slightly worse than the baseline, but much closer than any of the alternatives" that shrink h or the head dimension. It is orthogonal to local attention.

## Limitations

- Small models (192–211M) and short sequences (128–256).
- Fixed-shape padding.
- The configs were "to be added before publication".

> [!warning] Contradiction: how much quality MQA loses
> Here the loss is small: −0.2 dev BLEU and +0.3 LM perplexity, with beam-4 test BLEU actually higher. [[deepseek-v2-deepseek-ai-2024|DeepSeek-V2]]'s Table 8 (7B dense decoders, 1.33T tokens) instead finds large losses for MQA against MHA: MMLU 37.9 vs 45.2 and C-Eval 30.0 vs 42.9. Scale, task (knowledge benchmarks vs translation BLEU) and architecture all differ, so the cost of MQA is regime-dependent, not settled. See [[multi-query-attention]].

## Related

[[multi-query-attention]] · [[grouped-query-attention]] · [[grouped-query-attention-ainslie-2023]] · [[kv-cache]] · [[multi-head-attention]] · [[noam-shazeer]]
