---
title: Multi-Head Latent Attention
type: concept
created: 2026-09-29
updated: 2026-09-30
sources:
  - deepseek-v3.2-sparse-attention-deepseek-2025.md
  - native-sparse-attention-yuan-2025.md
  - misa-mixture-of-indexer-sparse-attention-zhou-2026.md
  - kimi-linear-kimi-team-2025.md
  - normalized-low-rank-adaptation-kang-2026.md
status: draft
importance: medium
---

# Multi-Head Latent Attention

**Multi-Head Latent Attention (MLA)** is the attention variant introduced with DeepSeek-V2. Instead of caching full per-head keys and values, it compresses them into a shared low-rank **latent vector** and caches that. The DeepSeek-V2 abstract credits MLA with cutting the [[kv-cache]] by 93.3% relative to DeepSeek 67B. Like [[grouped-query-attention]], it trades some of the per-head independence of [[multi-head-attention]] for a smaller decode-time memory footprint, but it gets there by compressing keys and values rather than by sharing them across heads.

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
