---
title: "GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - grouped-query-attention-ainslie-2023.md
arxiv_id: "2305.13245"
year: 2023
venue: EMNLP 2023
authors:
  - Joshua Ainslie
  - James Lee-Thorp
  - Michiel de Jong
  - Yury Zemlyanskiy
  - Sumit Sanghai
tags:
  - 2023
  - paper
status: complete
importance: high
---

# GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints

**Joshua Ainslie, James Lee-Thorp, Michiel de Jong (equal contribution), Yury Zemlyanskiy, Federico Lebrón, Sumit Sanghai** (Google Research; de Jong also University of Southern California) — [arXiv:2305.13245](https://arxiv.org/abs/2305.13245), accepted at **EMNLP 2023** (per the arXiv comments field).

## Summary

The paper's diagnosis is explicitly about **autoregressive decoding**, not prefill: "Autoregressive decoder inference is a severe bottleneck for Transformer models due to the memory bandwidth overhead from loading decoder weights and all attention keys and values at every decoding step." Multi-query attention (MQA, Shazeer 2019) fixes this by collapsing H key/value heads into one, shrinking the [[kv-cache]] — and hence the bytes that must be moved per decoded token — by a factor of H. But MQA "can lead to quality degradation and training instability," and most released checkpoints (T5, LLaMA) are multi-head, so you would have to train a separate model just to get fast inference.

Two contributions. **Uptraining**: convert an existing MHA checkpoint by **mean-pooling** the key and value projection matrices of each group down to one head, then continue pre-training on the original recipe and dataset for a small proportion α of the original pre-training steps. α = 0.05 — **5% of original pre-training compute** — suffices, costing roughly 600 TPUv3 chip-days at T5-XXL scale, with diminishing returns past 10%. Mean-pooling beats selecting a single head, which beats random re-initialization, "ordered by the degree to which information is preserved."

**Grouped-query attention** divides the H query heads into G groups, each sharing one key head and one value head, interpolating between the extremes: GQA-1 = MQA, GQA-H = MHA. The argument for an intermediate G is that as models get larger they add heads, so MQA becomes "a more aggressive cut in both memory bandwidth and capacity," while GQA keeps the proportional reduction constant with scale; GQA also removes the waste from standard sharding, which replicates MQA's single KV head across every model partition. At T5-XXL with G = 8 and α = 0.05, GQA lands at 47.1 average quality versus MHA-XXL's 47.2 and MQA's 46.6, at 0.28 per-sample inference time versus MHA-XXL's 1.51 and MQA's 0.24 — nearly MHA quality at nearly MQA speed.

> [!note] What GQA actually accelerates
> GQA is a **decode / memory-bandwidth** win, not a prefill-FLOPs win, and the paper is explicit about this. It is applied to decoder self-attention and cross-attention but deliberately **not to encoder self-attention**, because "encoder representations are computed in parallel, and memory bandwidth is therefore generally not the primary bottleneck." Prompt processing has exactly that parallel, compute-bound character. GQA reduces the *size of the KV cache that must be loaded per decoding step*; it does not restructure the attention computation itself. That is [[flash-attention]]'s job, and the two are complementary — the GQA paper cites FlashAttention as separate related work, and FlashAttention-2 in turn implements GQA by index manipulation rather than duplicating KV heads.

## Key points

- **GQA-G interpolates:** H query heads split into G groups sharing one K head and one V head each. **GQA-1 ≡ MQA**; **GQA-H ≡ MHA**. G = 8 chosen "as a favorable middle ground."
- **Uptraining cost: α = 0.05, i.e. 5% of original pre-training compute** (~600 TPUv3 chip-days for T5-XXL). Both MQA and GQA "gain from 5% uptraining with diminishing returns from 10%." GQA "already achieves reasonable performance after conversion while MQA requires uptraining to be useful."
- **Checkpoint conversion:** mean-pool the K/V projection matrices within each group. Ablation ordering (T5-Large → MQA, α = 0.05): **Mean > First > Random**.
- **Main results** (T5.1.1, average dev performance over CNN/DailyMail, arXiv, PubMed, MediaSum, MultiNews, WMT14 En-De, TriviaQA; T_infer is time per sample per TPUv4 chip, as labeled in Table 1 — note Figure 3 labels the same axis in ms):

| Model | T_infer | Average |
|---|---|---|
| MHA-Large | 0.37 | 46.0 |
| MHA-XXL | 1.51 | 47.2 |
| MQA-XXL (5% uptrained) | **0.24** | 46.6 |
| GQA-8-XXL (5% uptrained) | 0.28 | **47.1** |

  GQA-8-XXL is ~**5.4× faster** than MHA-XXL at 0.1 average points of quality loss, and beats MHA-Large on both axes.
- **Per-task (GQA-8-XXL vs. MHA-XXL):** CNN 43.5 / 43.8 R1, arXiv 45.4 / 45.6, PubMed 47.7 / 47.5, MediaSum 36.3 / 36.4, MultiNews 47.2 / 46.9, WMT 28.4 / 28.4 BLEU, TriviaQA 81.6 / 81.9 F1. GQA matches or beats MHA-XXL on PubMed and MultiNews.
- **Group count vs. speed (Fig. 6, input length 2048, output length 512):** going from 1 group (MQA) to 8 "adds modest inference overhead, with increasing cost as we move closer to MHA." The knee is why 8 was picked.
- **Why GQA scales better than MQA:** KV cache scales with model dimension while FLOPs and parameters scale with its square, so bandwidth pressure eases at larger sizes; meanwhile head count grows, making MQA's H-fold cut increasingly aggressive relative to capacity. GQA holds the ratio fixed. Sharding also replicates MQA's lone KV head per partition, which GQA avoids.
- **Not applied to encoder self-attention** — parallel-computed representations are not memory-bandwidth bound.
- **Stated limitations:** evaluated only on encoder–decoder (T5) models; no comparison against a GQA model trained from scratch, so uptraining-vs-scratch is unmeasured; Rouge is acknowledged as a flawed metric for the long-generation regime the method targets. The authors expect GQA's advantage over MQA to be **stronger** in decoder-only models, which lack the separate self-/cross-attention split.
- **Concurrent work:** Rabe (2023) "independently developed GQA with public implementation." The uptraining procedure is inspired by Komatsuzaki et al. (2022), which uptrains T5 checkpoints into sparse [[mixture-of-experts]] models.

## Concepts & entities

- [[grouped-query-attention]] — this is the origin paper for the technique now standard in decoder-only LLMs.
- [[kv-cache]] — the direct target: G key/value heads instead of H shrinks the cache and the bytes loaded per decoding step by H/G.
- [[attention-mechanism]] — GQA changes the head structure of attention (how many K/V heads exist), unlike FlashAttention which changes only its execution schedule.
- [[transformer-architecture]] — applied to decoder self-attention and cross-attention of T5.1.1, explicitly not to encoder self-attention.
- [[flash-attention]] — cited as complementary related work: FlashAttention "structures the attention computation to avoid materializing the quadratic attention scores," a distinct axis from reducing KV head count.
- [[quantization]] — cited as an orthogonal way to shrink keys and values (and weights) by lowering precision; the paper lists it alongside distillation, layer-sparse cross-attention, and speculative sampling as alternative attacks on the same decode memory-bandwidth bottleneck.
- [[mixture-of-experts]] — the uptraining recipe is adapted from sparse-upcycling work that converts dense T5 checkpoints into MoE models.

## References

- [arXiv:2305.13245](https://arxiv.org/abs/2305.13245)
