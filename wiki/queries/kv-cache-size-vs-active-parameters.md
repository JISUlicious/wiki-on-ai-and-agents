---
title: "How KV-cache size relates to (active) parameter count"
type: query
created: 2026-10-23
updated: 2026-10-23
sources:
  - quantization-performance.md
  - grouped-query-attention-ainslie-2023.md
  - kimi-linear-kimi-team-2025.md
  - qwen3.8-flash-next-model-card-2026.md
status: complete
importance: high
---

# How KV-cache size relates to (active) parameter count

**Question:** How does [[kv-cache]] size relate to a model's active parameter count? Explain with equations.

**Short answer:** the cache does not scale with active parameters as a whole. It scales with one slice of them, the **key/value projection weights, divided by the model width**.
- In dense multi-head attention that slice is a fixed share of all parameters (about 1/6).
- [[grouped-query-attention|GQA]], [[multi-head-latent-attention|MLA]], [[mixture-of-experts|MoE]] and hybrid linear attention each cut the link further.

## 1. The cache formula

```
c_tok = Σ_{attention layers} 2 · n_kv · d_h        (cached elements per token)
KV    = c_tok · T · B · b_kv                       (bytes)
```

- `n_kv` is the number of KV heads and `d_h` the head dimension.
- `T` is context length, `B` batch (concurrent sequences), and `b_kv` bytes per element.

This is the formula in [[quantization-performance]], which notes the cache "does not depend on parameter count" directly.

## 2. The exact link to parameters

Per layer, `W_K, W_V ∈ ℝ^(d × n_kv·d_h)`, so `P_KV,layer = 2·d·n_kv·d_h`. Summing over layers:

```
c_tok = P_KV / d
```

**Cached elements per token = K/V projection parameters ÷ model width.** This holds for any design that caches projection outputs; for MLA, use the down-projection whose latent output is cached. Query and output projections, FFN, experts and embeddings add nothing to the cache.

## 3. Dense multi-head attention

With MHA, `n_kv·d_h = d`, and a textbook dense block has `P ≈ 12·L·d²`:

```
c_tok = 2·L·d
c_tok / P ≈ 1/(6d)
T* = P / c_tok ≈ 6d
```

`T*` is the context length at which the cache, at batch 1 and the same precision, equals the weights. So the cache per token grows with `P/d`: proportional to parameters, but sublinearly, because `d` grows with model size. The [[grouped-query-attention-ainslie-2023|GQA paper]] makes the same point: "KV cache scales with model dimension while FLOPs and parameters scale with its square".

## 4. GQA divides the cache by H/G

| Model | `c_tok` = 2·L·n_kv·d_h | KV / token (FP16) | Params | `T*` = P / c_tok |
|---|---|---|---|---|
| Llama-2-7B (MHA) | 2·32·32·128 = 262,144 | 512 KiB | 6.74B | ≈ 25.7K |
| [[llama-3\|Llama-3]]-8B (GQA-8) | 2·32·8·128 = 65,536 | 128 KiB | 8.03B | ≈ 122K |
| Llama-3-70B (GQA-8) | 2·80·8·128 = 163,840 | 320 KiB | 70.6B | ≈ 431K |

- **The Llama-2-7B check matches the wiki:** 12.55 GiB of weights ÷ 512 KiB ≈ 25.7K tokens. So the cache passes the weights before 32K ([[kv-cache]]).
- **Bigger models have relatively smaller caches:** from 8B to 70B, parameters grow 8.8× while the cache grows 2.5×.

## 5. MoE: experts never enter the cache

```
P_active = P_attn + P_shared + k·P_expert + P_embed
c_tok    = P_KV / d                     ← attention slice only
```

Two MoE models with the same attention configuration have the same cache, whether they activate 3B or 40B of experts per token. **Active parameters predict compute, not cache size.**

## 6. Designs that shrink `c_tok` further

| Design | Cached elements per token | Example |
|---|---|---|
| [[multi-head-latent-attention\|MLA]] | `L·(d_c + d_R)`: a shared latent plus a small decoupled RoPE key | [[deepseek-v2-deepseek-ai-2024\|DeepSeek-V2]]: d_c = 512, d_R = 64, 60 layers → 34.6K elements/token, which is 4% of an MHA MoE of the same size (Table 9). The headline −93.3% is against a different dense model. |
| Hybrid linear attention | only the global layers cache per token; [[linear-attention]] layers hold a fixed state of `O(H·d_k·d_v)`, independent of `T` | [[kimi-linear-kimi-team-2025\|Kimi Linear]] (3:1): up to −75% KV |
| GQA + hybrid | `c_tok = 2·12·2·256 = 12,288` (24 KiB/token, bf16) | [[qwen3.8-flash-next\|Qwen3.8-Flash-Next]]: 12 global layers, 2 KV heads, head dim 256; 6B active → `T* ≈ 488K`, against ≈ 122K for dense Llama-3-8B |

The Qwen3.8 figures are computed here from the configuration recorded on its page. The 36 Gated DeltaNet layers add a fixed-size state on top. Its sparse attention limits what is *read* to about 2K tokens, but the cache is presumably still stored in full.

## 7. Two budgets, two parameter counts

**Memory capacity:**
```
M ≈ b_w·P_total + b_kv·c_tok·T·B
```
For MoE, every expert is resident, so the weights term uses **total** parameters.

**Decode bandwidth per step (batch 1):**
```
bytes/step ≈ b_w·P_active + b_kv·c_tok·T
KV reads exceed weight reads when   T > (b_w·P_active) / (b_kv·c_tok)
```
Here **active** parameters are the right measure. MoE keeps `P_active` small without shrinking `c_tok`, so MoE models become bound by KV reads at shorter contexts than a dense model of the same total size.

**Levers:**
- **KV quantization** lowers `b_kv`. A 4-bit cache against 16-bit weights moves the crossover 4× later ([[quantization-performance]]).
- **Batching** multiplies the cache term by `B` while the weights are shared, so KV memory usually caps serving concurrency. KIVI's throughput gain comes from fitting a larger batch.

For batched MoE, different tokens hit different experts, so the weight bytes read per step climb from `P_active` toward `P_total`.

## Summary

```
KV bytes = (P_KV / d) · T · B · b_kv
```

- **Dense multi-head attention:** `P_KV ≈ P/6`, so `T* ≈ 6d`.
- **GQA:** divides the cache by H/G.
- **MLA:** replaces K and V with a small latent.
- **Hybrid linear attention:** removes most layers from the cache.
- **MoE:** adds active parameters that never touch the cache.

## Gaps

- **Resolved 2026-10-23:** MLA now has its primary source, [[deepseek-v2-deepseek-ai-2024]].
- **Approximation:** `P ≈ 12Ld²` is the textbook dense estimate. The `T*` table uses real parameter counts instead.
- **Not modelled here, but now in the wiki:**
  - [[sliding-window-attention]] caps T per layer at W;
  - [[cross-layer-kv-sharing]] (CLA, YOCO) removes layers from the sum;
  - [[kv-cache-eviction]] (StreamingLLM, H2O, SnapKV) shrinks T;
  - [[kv-cache-quantization]] (KIVI, KVQuant) lowers b_kv;
  - [[jamba]] makes the layer-ratio point: 4 GB vs Mixtral's 32 GB at similar active parameters.

## References

- [[kv-cache]] · [[quantization-performance]] · [[grouped-query-attention]] · [[grouped-query-attention-ainslie-2023]] · [[multi-head-latent-attention]] · [[mixture-of-experts]] · [[linear-attention]] · [[kimi-linear-kimi-team-2025]] · [[qwen3.8-flash-next]] · [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]] · [[paged-attention]]
