---
title: "DeepSeek-V2: A Strong, Economical, and Efficient Mixture-of-Experts Language Model"
type: source
created: 2026-10-23
updated: 2026-10-23
sources:
  - deepseek-v2-deepseek-ai-2024.md
tags:
  - 2024
  - paper
status: complete
importance: high
authors:
  - DeepSeek-AI
introduces:
  - [[deepseek-v2]]
  - [[multi-head-latent-attention]]
year: 2024
venue: arXiv preprint (technical report)
arxiv_id: "2405.04434"
---

# DeepSeek-V2

[arXiv:2405.04434](https://arxiv.org/abs/2405.04434) (v5, 2024-06-19). Authored as "DeepSeek-AI"; more than 100 contributors are listed in the appendix. This is the **primary source for [[multi-head-latent-attention|Multi-head Latent Attention (MLA)]]**.

## Summary

[[deepseek-v2|DeepSeek-V2]] from [[deepseek]] is a **236B-total / 21B-active** [[mixture-of-experts|MoE]] model with a **128K** context, pretrained on 8.1T tokens. It combines two architecture pieces:
- **MLA**, which caches a compressed latent instead of full per-head K/V;
- **DeepSeekMoE**, with 2 shared + 160 routed experts, 6 routed experts active per token, and device-limited routing (each token goes to at most 3 of 8 devices).

## MLA: how the cache shrinks

Joint low-rank compression of keys and values:
```
c_t^KV = W^DKV h_t          (d_c ≪ n_h·d_h)
k_t^C  = W^UK c_t^KV,   v_t^C = W^UV c_t^KV
```
- **Only `c_t^KV` is cached.** `W^UK` can be absorbed into `W^Q` and `W^UV` into `W^O`, so full K/V are never materialized.
- **Decoupled RoPE key.** RoPE would block that absorption, so a small shared key `k_t^R = RoPE(W^KR h_t)` is added and cached too.

**Paper's Table 1 (cached elements per token):**

| Mechanism | KV cache per token | Capability (paper's label) |
|---|---|---|
| MHA | `2·n_h·d_h·l` | Strong |
| GQA | `2·n_g·d_h·l` | Moderate |
| MQA | `2·d_h·l` | Weak |
| **MLA** | **`(d_c + d_h^R)·l ≈ (9/2)·d_h·l`** | **Stronger** |

"d_c is set to 4d_h and d_h^R is set to d_h/2. So, its KV cache is equal to GQA with only 2.25 groups."

**Configuration:**
- 60 layers, hidden size 5120, n_h = 128, d_h = 128;
- **d_c = 512, d_h^R = 64**; query compression 1536.

That gives **(512 + 64) × 60 = 34,560 elements per token**, which matches Table 9's "34.6K". Against 1,966,080 for MHA with the same shape, that is about **1.8%**.

## Results

- **Against DeepSeek 67B** (abstract): "saves 42.5% of training costs, reduces the KV cache by 93.3%, and boosts the maximum generation throughput to 5.76 times".
  > [!note] What the headline compares
  > The 93.3% is against a **different dense GQA model** (DeepSeek 67B), not MLA against MHA on one model. The 5.76× throughput also includes **FP8 weights and KV-cache quantization to about 6 bits** per element. It isn't MLA alone.
- **The like-for-like comparison** (Table 9: MoE with MHA vs MLA):

  | | Small MHA | Small MLA | Large MHA | Large MLA |
  |---|---|---|---|---|
  | KV elements / token | 110.6K | 15.6K | 860.2K | 34.6K |
  | MMLU | 48.7 | 50.0 | 57.5 | 59.0 |
  | BBH | 37.9 | 39.0 | 46.6 | 50.7 |

  MLA needs 14% (small) and 4% (large) of MHA's cache **while scoring higher**.
- **Head-sharing ablation** (Table 8: 7B dense, 1.33T tokens; MQA / GQA-8 / MHA):
  - MMLU 37.9 / 41.2 / 45.2
  - C-Eval 30.0 / 37.7 / 42.9

  These MQA and GQA losses are much larger than in [[fast-transformer-decoding-one-write-head-shazeer-2019|Shazeer 2019]]; see [[multi-query-attention]].
- **Serving:** above 50K tokens/s generation on one 8×H800 node; prompt throughput above 100K tokens/s.
- **Benchmarks:** MMLU 78.5 for the base model. The chat (RL) model scores AlpacaEval 2.0 length-controlled 38.9 and MT-Bench 8.97.
- **Long context:** 4K → 128K via YaRN applied only to the decoupled RoPE key.
- **DeepSeek-V2-Lite:** 15.7B total / 2.4B active, 27 layers, d_c = 512.

## Limitations (stated)

- No knowledge updates after pretraining; hallucination remains.
- Mostly Chinese and English data.
- Text only.

## Related

[[multi-head-latent-attention]] · [[deepseek-v2]] · [[deepseek-v3]] · [[deepseek]] · [[mixture-of-experts]] · [[kv-cache]] · [[grouped-query-attention]] · [[multi-query-attention]] · [[kv-cache-size-vs-active-parameters]]
