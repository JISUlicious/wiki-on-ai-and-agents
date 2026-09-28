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

**GQA** interpolates between [[multi-head-attention|multi-head attention]] (MHA) and multi-query attention (MQA): the H query heads are split into **G groups**, each group sharing a single key/value head. `GQA-1` ≡ MQA, `GQA-H` ≡ MHA. Introduced by [[grouped-query-attention-ainslie-2023|Ainslie et al. (EMNLP 2023)]], it is now near-universal in production LLMs.

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

> [!warning] A cost that shows up only after context extension
> The table above prices GQA on quality-at-fixed-context and inference time. [[cracks-in-the-foundation-long-context-extension-bertsch-2026|Bertsch et al. (COLM 2026)]] price a different axis and find GQA is **not free for long-context extensibility**: on a controlled sweep of 26 comparable 7B models (38 checkpoints each, ~170,000 H100 hours), GQA-8 versus MHA-32 costs **−3.4 HELMET points at 32K after context extension**, and **−9 in combination with sliding-window attention**.
>
> GQA is one of four seemingly minor choices — with QK-norm (−3.8 to −6), pretraining context length (−1 to −2), and SWA (−1.1 alone) — that each ship in at least one of the Olmo, Llama and Qwen families. **Combining three or more drops downstream long-context performance by up to 47% relative** (26.5 points on HELMET), and the best single predictor of the damage is simply the *count* of features present (R² = 0.67).
>
> The part that matters operationally: **none of this is visible in loss.** End-of-extension loss explains R² = 0.06 of downstream score, held-out perplexity 0.01–0.39, and pre-extension HELMET at 8K only 0.32. A team choosing GQA on short-context evals has no signal that it is buying a long-context cost.
>
> This qualifies rather than contradicts the result above — Ainslie's decode-bandwidth claim stands untouched. Read the entry as "a trade whose cost appears only in post-extension long-context evaluation," not "an unambiguous win." Note also that Llama 3's configuration (GQA-8, but no QK-norm, no SWA, 8K pretraining) is among the *better* configs; the damage is combinatorial, not attributable to GQA alone.

- [[cross-model-kv-cache-transfer]] — 'matched-KV' pairs are defined by shared KV head count and per-head dim — GQA's parameters become the compatibility contract

## Related

- [[kv-cache]] — what GQA shrinks; the memory consumer that scales with context length.
- [[flash-attention]] — complementary and often combined; FA2 implements GQA/MQA by index manipulation rather than duplicating KV heads.
- [[attention-mechanism]] · [[minimax-sparse-attention-lai-2026]] (builds its blockwise sparse selection *on top of* GQA).

## References

- [[grouped-query-attention-ainslie-2023]]

## Related 2026 sources

- [[qwen3.8-flash-next]] — 24 Q : 2 KV with only 64 of 256 head dims rotary — an aggressive instance of the GQA-plus-hybrid profile Bertsch et al. flag as costly under context extension
