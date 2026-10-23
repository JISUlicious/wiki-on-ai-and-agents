---
title: Multi-Head Latent Attention
type: concept
created: 2026-09-29
updated: 2026-10-23
sources:
  - deepseek-v3.2-sparse-attention-deepseek-2025.md
  - native-sparse-attention-yuan-2025.md
  - misa-mixture-of-indexer-sparse-attention-zhou-2026.md
  - kimi-linear-kimi-team-2025.md
  - normalized-low-rank-adaptation-kang-2026.md
  - deepseek-v2-deepseek-ai-2024.md
status: draft
importance: medium
---

# Multi-Head Latent Attention

**Multi-Head Latent Attention (MLA)** is the attention variant introduced with DeepSeek-V2. Instead of caching full per-head keys and values, it compresses them into a shared low-rank **latent vector** and caches that. The primary source is [[deepseek-v2-deepseek-ai-2024|DeepSeek-V2]] ([[deepseek-v2]]), whose abstract credits it with cutting the [[kv-cache]] by 93.3% relative to DeepSeek 67B.
- **That 93.3% compares different models:** DeepSeek 67B is a separate dense GQA model.
- **The like-for-like result is in its Table 9:**
  - MLA caches **4% (large) and 14% (small)** of what MHA does in the same MoE, while scoring *higher* on MMLU and BBH.
  - Per token per layer, MLA caches the latent `d_c = 512` plus a decoupled RoPE key `d_h^R = 64`, about `(9/2)·d_h`, "equal to GQA with only 2.25 groups".
  - Over 60 layers that is 34.6K elements per token.
- **No full K/V needed:** the up-projections are absorbed into the query and output projections, so full K/V are never materialized. Like [[grouped-query-attention]], it trades some of the per-head independence of [[multi-head-attention]] for a smaller decode-time memory footprint, but it gets there by compressing keys and values rather than by sharing them across heads.

## In this wiki

- **Sparse attention runs on top of it.** [[deepseek-v3.2-sparse-attention-deepseek-2025|DeepSeek Sparse Attention]] runs under MLA's **MQA mode**, so each of the top-2,048 selected latent vectors is shared across all query heads. That sharing is what makes gathering 2,048 entries per query affordable. [[misa-mixture-of-indexer-sparse-attention-zhou-2026|MiSA]] retrofits the indexer of that same sparse MLA in DeepSeek-V3.2 and GLM-5.
- **GQA vs MLA as the kernel substrate.** [[native-sparse-attention-yuan-2025|NSA]] is built on GQA and DSA on MLA. Its source page notes that this difference matters for the kernel. See [[trained-sparse-attention]].
- **Full-attention baseline in hybrids.** [[kimi-linear-kimi-team-2025|Kimi Linear]] interleaves KDA layers with full MLA layers at 3:1 and uses pure MLA as its baseline throughout, for its scaling law, its efficiency figures and RULER at 128K (MLA 81.3 vs Kimi Linear 84.3).
- **Source of a fine-tuning idea.** [[normalized-low-rank-adaptation-kang-2026|NoRA]] takes MLA's normalized latent bottleneck `B·Norm(Ax)` and turns it into a mergeable [[lora]] initialization.

## Related

- [[multi-head-attention]] · [[grouped-query-attention]] · [[kv-cache]] · [[sparse-attention]] · [[trained-sparse-attention]] · [[deepseek]] · [[deepseek-v3]] · [[mixture-of-experts]]

## References

- DeepSeek-AI, "DeepSeek-V2: A Strong, Economical, and Efficient Mixture-of-Experts Language Model" (2024), [arXiv:2405.04434](https://arxiv.org/abs/2405.04434). This paper introduced MLA.
- [[deepseek-v3.2-sparse-attention-deepseek-2025]] · [[native-sparse-attention-yuan-2025]] · [[misa-mixture-of-indexer-sparse-attention-zhou-2026]] · [[kimi-linear-kimi-team-2025]] · [[normalized-low-rank-adaptation-kang-2026]]

## Related 2026 sources

- [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]] — DeepSeek-V4.1-Flash: encoder–decoder prefill split, cross-layer sparse KV reuse, FP4 KV; indexer cut to 32 heads, top-k 512
- [[memory-attention-kang-2026]] — Memory Attention: values = keys + a per-layer token-ID embedding, no value projection; tables offloadable to CPU

## Lineage sources

- [[kv-cache-size-vs-active-parameters]] — MLA caches `L·(d_c + d_R)` per token. The same `c_tok = P_proj/d` identity holds for the latent down-projection
