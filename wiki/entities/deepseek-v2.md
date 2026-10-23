---
title: DeepSeek-V2
type: entity
created: 2026-10-23
updated: 2026-10-23
sources:
  - deepseek-v2-deepseek-ai-2024.md
tags:
  - model
status: draft
importance: medium
family: deepseek
successors:
  - [[deepseek-v3]]
parameters: 236B total / 21B active
release_date: 2024-05-07
---

# DeepSeek-V2

[[deepseek]]'s May 2024 [[mixture-of-experts|MoE]] model (**236B total / 21B active**, 128K context, 8.1T pretraining tokens). It **introduced [[multi-head-latent-attention|Multi-head Latent Attention (MLA)]]** and DeepSeekMoE, which [[deepseek-v3|DeepSeek-V3]] kept. Paper: [[deepseek-v2-deepseek-ai-2024]].

## Key facts

- **MLA:** caches a 512-dimensional latent plus a 64-dimensional RoPE key per layer. Over 60 layers that is **34.6K elements per token**, about 4% of an MHA MoE of the same size, with *higher* MMLU and BBH.
- **Against DeepSeek 67B:** "reduces the KV cache by 93.3%" and throughput up to **5.76×**. The comparison is against a different dense GQA model, and the throughput figure includes FP8 weights and KV quantization to about 6 bits.
- **DeepSeekMoE:** 2 shared + 160 routed experts, 6 routed experts active per token, device-limited routing.
- **DeepSeek-V2-Lite:** 15.7B / 2.4B active.

## Lineage

DeepSeek 67B (dense, GQA) → **DeepSeek-V2**, which introduces MLA and DeepSeekMoE → [[deepseek-v3]] (671B / 37B; same MLA dimensions per its released config, not a wiki source). V3 is the base of [[deepseek-r1]], and V3.2 and V4.1 later add sparse attention and further KV compression ([[deepseek-v3.2-sparse-attention-deepseek-2025]], [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]]).

## References

- [[deepseek-v2-deepseek-ai-2024]]
