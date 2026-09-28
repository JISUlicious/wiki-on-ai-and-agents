---
title: DeepSeek-V3
type: entity
created: 2026-09-29
updated: 2026-09-29
sources:
  - deepseek-r1-2025.md
  - scalable-moe-training-megatron-core-yan-2026.md
  - cross-family-speculative-prefill-upasani-2026.md
  - skillreducer-gao-2026.md
  - swe-bench-live-zhang-2025.md
family: deepseek
parameters: 671B total / 37B active
status: draft
importance: medium
tags:
  - model
---

# DeepSeek-V3

Large [[mixture-of-experts]] language model from [[deepseek|DeepSeek]], described in a technical report of December 2024. It has **671B total parameters with 37B activated per token** and was pre-trained on 14.8T tokens. In this wiki it matters mainly as **the base model underneath [[deepseek-r1|DeepSeek-R1]]**, and secondly as a reference-scale MoE for training and inference systems work.

## In this wiki

- **Base of R1.** [[deepseek-r1-2025|DeepSeek-R1]] applies [[grpo|GRPO]] directly to DeepSeek-V3-Base with no SFT to get R1-Zero, whose AIME 2024 pass@1 rises from 15.6% to 77.9%. V3 supplies the parametric prior that the `*-R1` agentic-RL line ([[search-r1]], [[memory-r1]]) inherits by way of R1.
- **MoE training at scale.** [[scalable-moe-training-megatron-core-yan-2026|Megatron Core]] cuts DeepSeek-V3's footprint from 199.5 GB to under 80 GB per GPU with fine-grained recompute, BF16 optimizer moments and CPU activation offload. See [[training-memory-anatomy]].
- **Long-context serving target.** [[cross-family-speculative-prefill-upasani-2026|Cross-family speculative prefill]] runs RULER at 128K against a DeepSeek-V3 target capped at 32K on SambaNova's RDU ([[sambanova]]). That forced KV compression, so its "full prompt" row is not a clean dense baseline.
- **Workhorse / evaluated model.** DeepSeek-V3 is the compression classifier in [[skillreducer-gao-2026|SkillReducer]]. The -0324 checkpoint is one of four models on [[swe-bench-live-zhang-2025|SWE-bench-Live]].

## Lineage

Successor to DeepSeek-V2 (no page) in the [[deepseek]] line. Per its technical report, it adopts the [[multi-head-latent-attention|Multi-head Latent Attention (MLA)]] and DeepSeekMoE architectures first validated in V2, and scales them to 671B parameters. Descendants include the R1 reasoning models ([[deepseek-r1]]) and DeepSeek-V3.2, which adds trained [[sparse-attention]] (DSA, see [[deepseek-v3.2-sparse-attention-deepseek-2025]]).

## Related

- [[deepseek]] · [[deepseek-r1]] · [[mixture-of-experts]] · [[multi-head-latent-attention]] · [[grpo]] · [[trained-sparse-attention]]

## References

- [DeepSeek-V3 Technical Report, arXiv:2412.19437](https://arxiv.org/abs/2412.19437) — 671B / 37B active, 14.8T tokens, MLA + DeepSeekMoE from V2
- [[deepseek-r1-2025]] · [[scalable-moe-training-megatron-core-yan-2026]] · [[cross-family-speculative-prefill-upasani-2026]] · [[skillreducer-gao-2026]] · [[swe-bench-live-zhang-2025]]
