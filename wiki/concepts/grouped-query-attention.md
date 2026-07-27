---
title: Grouped-Query Attention (GQA)
type: concept
created: 2026-07-27
updated: 2026-07-27
sources:
  - grouped-query-attention-ainslie-2023.md
builds-on:
  - [[attention-mechanism]]
status: complete
importance: high
---

# Grouped-Query Attention (GQA)

**GQA** interpolates between multi-head attention (MHA) and multi-query attention (MQA): the H query heads are split into **G groups**, each group sharing a single key/value head. `GQA-1` ≡ MQA, `GQA-H` ≡ MHA. Introduced by [[grouped-query-attention-ainslie-2023|Ainslie et al. (EMNLP 2023)]], it is now near-universal in production LLMs.

## Why it exists

MQA shrinks the [[kv-cache|KV cache]] aggressively but degrades quality and can be unstable; MHA is expensive to serve. GQA recovers most of MHA's quality at close to MQA's speed:

| T5-XXL | Quality | T_infer |
|---|---|---|
| MHA | 47.2 | 1.51 |
| **GQA-8** | **47.1** | **0.28** |
| MQA | 46.6 | 0.24 |

**Uptraining** converts an existing MHA checkpoint rather than training from scratch: mean-pool the K/V projections of each group (this beats selecting the first head or random init), then train for **α = 5% of original pre-training compute**, with diminishing returns past 10%.

> [!warning] What GQA actually accelerates — not prefill
> GQA targets *"the memory bandwidth overhead from loading decoder weights and all attention keys and values **at every decoding step**"* — a **decode** bottleneck. The paper applies it to decoder self-attention and cross-attention, and **deliberately not to encoder self-attention**, on the explicit grounds that *"encoder representations are computed in parallel, and memory bandwidth is therefore generally not the primary bottleneck."*
>
> That parallel-computation regime **is prefill**. So GQA is a KV-cache/decode optimization, and should not be expected to cut prompt-processing FLOPs. See [[quantization-performance]] for the general asymmetry.

Its prefill-adjacent benefit is indirect but real: a smaller KV cache frees HBM, raising the batch size and context length you can serve at all.

## Related

- [[kv-cache]] — what GQA shrinks; the memory consumer that scales with context length.
- [[flash-attention]] — complementary and often combined; FA2 implements GQA/MQA by index manipulation rather than duplicating KV heads.
- [[attention-mechanism]] · [[minimax-sparse-attention-lai-2026]] (builds its blockwise sparse selection *on top of* GQA).

## References

- [[grouped-query-attention-ainslie-2023]]
