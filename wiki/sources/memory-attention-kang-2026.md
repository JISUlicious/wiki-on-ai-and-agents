---
title: "Memory Attention"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - memory-attention-kang-2026.md
arxiv_id: "2609.28399"
authors:
  - Jiale Kang
year: 2026
status: complete
importance: low
tags:
  - 2026
  - paper
---

# Memory Attention

**Jiale Kang** (single author; no affiliation given) — [arXiv:2609.28399](https://arxiv.org/abs/2609.28399), cs.LG, v1 23 Sep 2026.

## Summary

Memory Attention (MA) removes the value projection W_V entirely. Values are built as **V = K + RMSNorm(E_ℓ[token_id])**, where E_ℓ is a vocabulary-sized embedding table specific to each layer. The keys (taken before RoPE) carry the contextual information, and the table carries information tied to the token. At inference, the normalization is folded into the table, so building a value costs only a lookup and an addition. Because the lookup address depends only on token ID and layer, the tables can be **offloaded to CPU and prefetched** (MA-Offload). The paper relates MA to the aggregation form of [[multi-head-latent-attention|MLA]]: both attend over a shared representation (K for MA, the latent for MLA), and MA replaces MLA's learned value map with an additive memory read. **MA-Recall**, which would rebuild V from cached K plus token IDs and so halve the content KV cache, is analysed but not measured.

## Key points

- **Scale.** Models have 24 layers, trained on H800 with flash-linear-attention. Hidden size 1,024 is trained on 10B tokens; hidden size 2,048 on 20B.
- **The comparison is not parameter-matched.** In Table 2, Standard vs MA is 373M vs 1,135M (small MHA), 349M vs 729M (GQA), and 1,364M vs 2,836M (large MHA). The author says plainly that the gains "do not isolate the contribution of its structure from the increase in parameter capacity".
- **Quality.**
  - WikiText PPL falls from 31.55 to 28.64 (small MHA), and LAMBADA PPL from 50.46 to 43.68 (GQA).
  - Average accuracy gains are **+0.71 / +0.61 / +1.16** points.
  - The large-model gains come from OpenBookQA (+3.00) and ARC-C (+2.65); ARC-E and PIQA drop slightly.
- **Token efficiency.** MA reaches matched losses with 1.42× (D1024) and 1.16× (D2048) fewer tokens, that is 29.6% and 13.8% fewer. These figures are token counts, not wall-clock time.
- **Needle retrieval (single-needle NIAH, 2K training window).** Mean accuracy is 97.4% vs 82.6% at 1K and 96.8% vs 82.1% at 2K. At a 4K extrapolation it is 41.9% vs 25.9%; both models collapse beyond the training window.
- **Inference (single H800, batch 8, 2K).**
  - Prefill takes 90.970 → 88.202 ms (−3.04%). Decode takes 17.636 → 17.788 ms (+0.86%).
  - MA-Offload moves 1,572.864M table parameters (3,000 MiB BF16) to CPU. GPU parameter storage drops from 5,410 to 2,410 MiB, which is 192 MiB (7.38%) below Standard, while holding about 2.08× the parameters.
  - With offload, measured latency is 90.686 / 17.189 ms.

## What it does not show

- Nothing larger than about 2.8B total parameters. SWA, linear-attention, larger and MoE backbones are listed as "ongoing".
- No parameter-matched baseline and no ablation against Value Embeddings or Engram at equal table size.
- MA-Recall's KV saving is only arithmetic. The KV cache in the latency runs is conventional.

## Concepts & entities

- [[multi-head-latent-attention]] — the formal comparison in §3.2.
- [[kv-cache]] — MA-Recall would store K only; the measured runs still store K and V.
- [[weight-streaming]] · [[activation-sparsity]] — MA-Offload is lookup-addressed parameter offload. Its prefetch list is exact, like MoE routing, because it is known from token IDs.
- [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]] — ships Engram (196B lookup parameters, prefetched from host memory by RDMA). That is the same "lookup-addressed capacity off-GPU" idea that MA applies to the value path.
- [[grouped-query-attention]] · [[multi-head-attention]] · [[needle-in-a-haystack]] · [[nvidia]] (H800).

## References

- [arXiv:2609.28399](https://arxiv.org/abs/2609.28399)
- Related token-indexed memory: Value Embedding; DeepEmbed; Per-Layer Embeddings (Gemma 3n); STEM; Engram (Cheng et al., "Conditional memory via scalable lookup").
- MLA: DeepSeek-V2/V3. Evaluation: lm-evaluation-harness; LAMBADA, WikiText, ARC, HellaSwag, PIQA, WinoGrande, OpenBookQA.
