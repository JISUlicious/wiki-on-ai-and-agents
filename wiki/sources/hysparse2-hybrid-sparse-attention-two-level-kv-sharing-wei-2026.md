---
title: "HySparse2: Hybrid Sparse Attention with Two-Level KV Sharing"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - hysparse2-hybrid-sparse-attention-two-level-kv-sharing-wei-2026.md
arxiv_id: "2609.26368"
authors:
  - Jianyu Wei
  - Yizhao Gao
  - Qihao Zhang
  - Shimao Chen
  - Zhengju Tang
  - Yu Cheng
  - Shengjie Zhou
  - Zihan Jiang
  - Yifan Song
  - Hailin Zhang
  - Liang Zhao
  - Bo Yang
  - Gang Wang
  - Shijie Cao
  - Fuli Luo
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# HySparse2: Hybrid Sparse Attention with Two-Level KV Sharing

**Wei, Gao (equal) … Cao, Luo** — LLM-Core, [[xiaomi|Xiaomi]]. [arXiv:2609.26368](https://arxiv.org/abs/2609.26368), v1 22 Sep 2026. No venue is stated.

## Summary

HySparse2 is the successor to HySparse (arXiv:2602.03560). It applies a **YOCO-style self-decoder / cross-decoder split** to a hybrid sparse-attention backbone. The self-decoder mixes full attention with SWA; the cross-decoder mixes full attention with sparse attention. There are two levels of KV sharing:

- **KV Bridging (outer level).** Each cross-decoder full-attention layer applies its own K/V projections to the input hidden states of a matching self-decoder full-attention layer.
- **KV Reuse (inner level, inherited from HySparse).** Sparse layers reuse the preceding full-attention layer's KV cache and its top-k selection.

Two refinements make every cross-decoder KV buildable from self-decoder states:

- **Token-level selection** replaces 64-token blocks.
- A **forced 128-token recent window** inside the sparse selection replaces the separate SWA branch.

As a result, prefill can **exit after the self-decoder**. For the 49-layer model, only the first 25 layers run, and only one of them is a full-attention layer. Full-attention layers double as oracle indexers, so the model has no separately trained indexer.

## Key points

- **Setup.** 80B-A3B MoE, 49 layers, hidden size 2,048. Pretraining on about 500B tokens at 32k, then light post-training on about 100B tokens with agentic data, extended to 256k. HySparse and HySparse2 each use **5 full-attention layers**; Hybrid SWA (the MiMo-V2 design) needs 9. HySparse2 uses MQA, while the baselines use GQA. SWA layers use partial RoPE; full and sparse layers use NoPE. The sparse budget is 128 forced + 1,024 global tokens.
- **Pretraining (Table 2).** General capability is broadly on par with the baselines. RULER is +5.88 and NoLiMa +9.49 over HySparse. Repo Code PPL is 1.1570 vs 1.1588 (HySparse) and 1.1578 (Hybrid SWA).
- **After post-training (Fig. 3).**
  - Mean MRCR-v2 is **+11.30** and RULER-v2 **+19.81** over HySparse; the margins over Hybrid SWA are +6.44 and +18.65.
  - At 256k, RULER-v2 is **58.45** vs 32.61 (HySparse) and 35.74 (Hybrid SWA).
  - AgentPPL and LongPPL are lower at every length.
- **Efficiency at 1M tokens (Fig. 4, analytical, FP8 KV).**
  - Prefill FLOPs are **2.92×** lower than HySparse and **5.02×** lower than Hybrid SWA.
  - The KV cache is **2.69 GB**, vs 6.72 GB (HySparse) and 12.09 GB (Hybrid SWA).
- **Token vs block selection (Table 3).** At the same budget, token-level selection improves RULER-v2 by +6.57, two-needle MRCR-v2 by +8.14 and GraphWalks by +5.55.
- **Forced vs gated SWA (Table 4).** Forced SWA gives the best RULER, RULER-v2 and GraphWalks scores. It is 5.08 points lower on GSM8K and 4.99 lower on MRCR-v2, and its LongPPL is 1.50% higher. The authors accept this cost because it removes the local KV cache.
- **KV Bridging at 290B-A8B (1.8T tokens, Table 5).**
  - Quality is broadly the same: RULER moves 0.31, and LongPPL improves from 3.6053 to 3.4202.
  - DROP falls from 71.37 to 68.17, and BBH and GSM8K each drop about 1 point.
  - Against the U-shaped KV Mirror, the final RULER is 87.65 vs 81.28.
- **Deployment.** Under prefill–decode disaggregation, the prefill node hosts only the self-decoder and the bridging projections, which is about half of the weights.

## What it does not show

- Prefill-FLOPs and KV-size figures are computed, not measured end to end. There are no latency or throughput numbers.
- Evaluation after post-training covers only retrieval and perplexity metrics. It uses internal AgentPPL and Repo Code PPL, and there is no agentic task success rate.
- The MQA vs GQA difference between arms confounds the KV-size comparison.

## Concepts & entities

- [[sparse-attention]] · [[trained-sparse-attention]] — token-level selection with full attention acting as the indexer, instead of a distilled lightning indexer.
- [[kv-cache]] · [[grouped-query-attention]] · [[multi-head-latent-attention]] — the paper frames KV compression along head, sequence, layer and precision axes; HySparse2 works on the layer axis.
- [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]] — a parallel YOCO-derived design. It keeps a decoder SWA branch and approximates it with bounded replay; HySparse2 removes that branch by construction.
- [[claa-cross-layer-attention-aggregation-mcdanel-2026]] · [[droidspeak-kv-cache-sharing-liu-2024]] — other cross-layer KV and selection sharing.
- [[mimo-v2-6-scaling-reinforcement-learning-self-improvement-xiaomi-2026]] · [[xiaomi]] — the MiMo line whose Hybrid SWA is the baseline here.
- [[ruler]] · [[needle-in-a-haystack]] · [[long-context-llm]] · [[attention-sink]] (learnable per-head sink biases) · [[positional-encoding]] (NoPE in full and sparse layers).
- [[distserve-zhong-2024]] · [[splitwise-patel-2023]] — the prefill–decode disaggregation setting the paper targets.

## References

- [arXiv:2609.26368](https://arxiv.org/abs/2609.26368)
- HySparse: Gao et al. 2026, arXiv:2602.03560. YOCO: Sun et al. 2024. KV Mirror: Liu et al. 2026. Cross-layer attention: Brandon et al., NeurIPS 2024.
- MiMo-V2-Flash: Xiaomi Team, arXiv:2601.02780. DeepSeek-V4.1-Flash (bounded replay): DeepSeek-AI 2026. DFlash: Chen et al., arXiv:2602.06036.
