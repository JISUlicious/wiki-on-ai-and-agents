---
title: "Token Sparse Attention: Efficient Long-Context Inference with Interleaved Token Selection"
type: source
created: 2026-09-03
updated: 2026-09-03
sources:
  - token-sparse-attention-jo-2026.md
arxiv_id: "2602.03216"
venue: ICML 2026
year: 2026
authors:
  - Dongwon Jo
  - Beomseok Kang
  - Jiwon Song
  - Jae-Joon Kim
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Token Sparse Attention: Efficient Long-Context Inference with Interleaved Token Selection

**Dongwon Jo, Beomseok Kang, Jiwon Song, Jae-Joon Kim** (Department of Electrical and Computer Engineering, Seoul National University; corresponding authors Beomseok Kang and Jae-Joon Kim) — [arXiv:2602.03216](https://arxiv.org/abs/2602.03216), cs.CL, v1 3 Feb 2026, v3 29 May 2026. **ICML 2026.** Code: `https://github.com/dongwonjo/Token-Sparse-Attention`. Same group as FastKV (Jo et al. 2025), the token-eviction method this paper positions itself against.

## Summary

Two families accelerate long-context prefill without retraining, and this paper argues both leave something on the table. *Block-sparse attention* ([[minference-dynamic-sparse-attention-jiang-2024|MInference]], FlexPrefill) skips regions of the attention map but must keep whole 64-token blocks to stay kernel-friendly, so uninformative tokens survive whenever they share a block with a salient one. *Token eviction* (FastKV, GemFilter, PyramidInfer) works at token granularity but decides once, in an early layer, and discards permanently — while the authors' own probe on LLaMA-3.1-8B-Instruct shows that the top-1% tokens overlap only moderately between adjacent layers and the overlap decays quickly with layer distance (Fig. 2a), and that different heads in the same layer (layer 18) rank tokens differently (Fig. 2b). A single, shared, irreversible token set therefore either over-keeps or throws away what a later layer or another head needed.

**Token Sparse Attention** is a *compress-then-decompress* wrapper around any attention kernel. Per head, it gathers the rows of Q, K and V for a selected token subset of size L′ ≪ L, so the compressed tensors are dense and contiguous and any unmodified kernel (FlashAttention, MInference, FlexPrefill, SeerAttention, X-Attention) runs attention in the L′×L′ space at cost O(L′²d). The output is then *scattered back* into a zero-initialised L×d tensor at the selected positions and added to the residual. Unselected tokens get a zero attention output for that head — equivalent to a hard mask — but their hidden state survives through the residual connection, so every layer re-selects from the full sequence and each head selects its own tokens. The budget is set online by **Dynamic Token Coverage**: a Triton kernel scores tokens per head by the column-sums of attention from the most recent queries (last_q), aggregates across heads, sorts *ascending*, and drops the least-important tail whose cumulative attention mass is at most a coverage threshold τ (τ = 0.005 for LLaMA, 0.008 for Mistral); each head then keeps its own top-k_keep. Because applying this everywhere hurts, a **Sparse Layer Selection** step computes each layer's *inter-layer representation drift* (relative L2 change of hidden states across the layer) once per model and sparsifies only the lower-drift half of layers (δ = 0.5), validated by 200 random 3-layer subsets where lower mean drift correlated with higher RULER accuracy.

Measured on LLaMA-3.1-8B-Instruct and Mistral-Nemo-12B-Instruct on one A100 80GB: at 128K, Token Sparse Attention lifts the *attention* speedup of FlashAttention to ×1.36, MInference from ×1.12 to ×1.38, FlexPrefill from ×2.44 to ×2.76, SeerAttention from ×2.19 to ×2.47 and X-Attention from ×2.72 to ×3.49, with RULER averages moving by at most ~0.5 points (FlashAttention 87.01 → 87.02; FlexPrefill 87.27 → 87.27). End-to-end prefill (TTFT) at 128K goes from 31.04 s to 24.35 s for FlashAttention (×1.27) and to 16.81 s for FlexPrefill + Token Sparse (×1.85 vs. dense). At a matched ~×1.5 attention speedup it beats the eviction methods on RULER (86.84 vs. FastKV 85.64, GemFilter 85.12, PyramidInfer 78.49). It is explicitly a *complement*: attention-map sparsity is only 54% at 128K (τ = 0.005), there is no TTFT gain at 8K (×0.97), and the abstract's "up to ×3.23" does not correspond to any table in the body (see below).

## Key points

**Where this sits: attention sparsification at token granularity — all tokens stay in the sequence.** Q, K, V projections are computed for all L tokens *before* the gather; the scattered output is L×d, so the output projection and the FFN run densely over all L tokens; and the KV cache is full. Only the O(L²) attention term is reduced (to O(L′²) per head in the selected layers). This is what separates it from *[[token-pruning|token pruning]]* (FastKV, GemFilter, PyramidInfer — and the non-evicting OrthoRank, which the paper notes "additionally skips FFN computation for unselected tokens", forcing the comparison in Table 13 onto *prefill* speedup) and from *[[speculative-prefill|speculative prefill]]* (a draft model choosing which tokens to drop; see [[claa-cross-layer-attention-aggregation-mcdanel-2026]]). Versus [[minference-dynamic-sparse-attention-jiang-2024|MInference]] the unit is a token per head rather than a 64×64 block or a column/diagonal, and the selection is reversible layer to layer. Because the wrapper produces dense tensors, it composes with the block-sparse kernels rather than replacing them — every headline number is a "baseline + Token Sparse" pair.

**Composition results — RULER, 4K–128K** (Table 1; speedup = attention-kernel speedup at 128K vs. FlashAttention, averaged over layers):

| Model | Method | 4K | 8K | 16K | 32K | 64K | 128K | Avg | Attn speedup @128K |
|---|---|---|---|---|---|---|---|---|---|
| LLaMA-3.1-8B-Instruct | FlashAttention | 95.82 | 92.77 | 91.02 | 84.87 | 83.43 | 74.15 | 87.01 | ×1.00 |
| | + Token Sparse | 96.06 | 92.90 | 91.82 | 84.81 | 82.83 | 73.68 | 87.02 | ×1.36 |
| | MInference (uniform Vertical-Slash) | 93.46 | 92.29 | 91.01 | 85.34 | 83.19 | 73.63 | 86.49 | ×1.12 |
| | + Token Sparse | 93.05 | 92.00 | 91.03 | 85.10 | 82.92 | 72.18 | 86.05 | ×1.38 |
| | FlexPrefill (γ = 0.95) | 95.48 | 92.71 | 91.40 | 87.20 | 83.05 | 73.75 | 87.27 | ×2.44 |
| | + Token Sparse | 95.33 | 92.47 | 91.49 | 87.68 | 83.07 | 73.58 | 87.27 | ×2.76 |
| Mistral-Nemo-12B-Instruct | FlashAttention | 95.19 | 92.29 | 85.60 | 64.86 | 47.98 | 19.68 | 67.60 | ×1.00 |
| | + Token Sparse | 95.07 | 92.10 | 85.97 | 63.67 | 47.97 | 19.44 | 67.37 | ×1.22 |
| | MInference | 92.52 | 91.02 | 84.29 | 65.18 | 46.52 | 19.00 | 66.42 | ×1.13 |
| | + Token Sparse | 93.01 | 91.36 | 84.31 | 64.70 | 46.67 | 18.68 | 66.46 | ×1.28 |
| | FlexPrefill | 94.79 | 93.13 | 86.62 | 64.58 | 49.30 | 20.54 | 68.16 | ×1.22 |
| | + Token Sparse | 94.84 | 93.31 | 86.14 | 64.89 | 48.70 | 19.58 | 67.91 | ×1.33 |

Additional kernels (Table 12, LLaMA-3.1-8B): SeerAttention 86.99 @ ×2.19 → 87.02 @ ×2.47; X-Attention (S = 16, τ = 0.9) 83.39 @ ×2.72 → 83.51 @ **×3.49**. Note the MInference baseline here is the paper's own configuration — a Vertical-Slash pattern applied uniformly to all heads, without MInference's per-head kernel-aware search — on LLaMA-3.1-8B rather than the Gradient 262K/1M checkpoints, which is why it shows only ×1.12 at 128K where the MInference paper reports 1.8× end-to-end at 100K.

**InfiniteBench and LongBench** (Tables 2, 6; averages): LLaMA-3.1-8B InfiniteBench — FlashAttention 50.86 → 50.88; MInference 50.16 → 49.70; FlexPrefill 49.53 → 49.23. Mistral-Nemo — 22.93 → 22.80; 19.77 → 19.41; 24.80 → 24.08. Retr.KV is the sensitive task: 55.20 → 55.00 on FlashAttention but 51.60 → 49.60 on MInference and 50.80 → 46.80 on FlexPrefill. LongBench (LLaMA-3.1-8B, 16 tasks): 49.28 → 48.96; 47.89 → 47.61; 47.48 → 47.39, with PassageCount the largest single drop (9.41 → 7.47).

**How much attention actually gets skipped** (Table 3; average attention-map sparsity in the selected layers):

| τ | 4K | 8K | 16K | 32K | 64K | 128K |
|---|---|---|---|---|---|---|
| 0.005 | 17.00% | 21.11% | 26.61% | 28.44% | 34.07% | 54.44% |
| 0.010 | 28.02% | 32.98% | 39.55% | 41.25% | 47.32% | 67.36% |

Sparsity grows with length because the "attention noise" tail below τ gets longer. These are far lower than the >90% kernel sparsity MInference reaches at 200K+, which is the honest reason this method is a complement: it cheaply removes the tail that block-sparse kernels keep. Overhead (scoring + indexing + gather + scatter) is under 11% of total attention latency at 128K at the highest sparsity tested.

**Dynamic coverage vs. a fixed keep ratio** (Table 4, RULER, LLaMA-3.1-8B, matched 128K speedup):

| | τ = 0.005 | τ = 0.010 | fixed s = 0.3 | fixed s = 0.5 |
|---|---|---|---|---|
| Sparsity | 54.44% | 67.36% | 50.96% | 74.95% |
| RULER avg | 87.02 | 86.84 | 86.91 | 85.43 |
| Attn speedup | ×1.36 | ×1.51 | ×1.32 | ×1.57 |

**Attention-kernel speedup vs. end-to-end prefill (TTFT)** (Tables 10–11; LLaMA-3.1-8B-Instruct, one GPU; TPOT is identical for all methods — 89 ms at 128K with 128 generated tokens — because decode is dense with a full KV cache):

| Method | A100 8K | A100 128K | A6000 8K | A6000 128K |
|---|---|---|---|---|
| FlashAttention | 0.68 s (×1.00) | 31.04 s (×1.00) | 1.76 s (×1.00) | 67.34 s (×1.00) |
| + Token Sparse | 0.70 s (×0.97) | 24.35 s (**×1.27**) | 2.04 s (×0.86) | 54.49 s (×1.24) |
| MInference | 1.52 s (×0.45) | 26.84 s (×1.16) | 4.00 s (×0.44) | 53.54 s (×1.26) |
| + Token Sparse | 1.53 s (×0.44) | 23.32 s (×1.33) | 3.71 s (×0.47) | 47.05 s (×1.43) |
| FlexPrefill | 0.90 s (×0.76) | 18.45 s (×1.68) | 3.22 s (×0.55) | 47.16 s (×1.43) |
| + Token Sparse | 0.92 s (×0.74) | 16.81 s (**×1.85**) | 2.68 s (×0.66) | 43.08 s (×1.56) |

The split is stark. At 128K on A100, a ×1.36 attention speedup becomes ×1.27 TTFT, and FlexPrefill's ×2.44 attention becomes ×1.68 TTFT. Solving the Amdahl relation for those pairs puts attention at roughly **70–80% of dense prefill time for LLaMA-3.1-8B at 128K on an A100** (derived here, not stated in the paper; the paper only says "attention dominates in long-context regimes"). The remaining 20–30% — the compute-bound projections and FFN at ~1,024–1,215 OPs/byte in the [[llm-inference-roofline-yuan-2024]] analysis — is untouched by every method in the table. At 8K, attention sparsity is only ~21% and the scoring overhead is visible, so TTFT is flat (×0.97) for this method and *worse than dense* for both block-sparse baselines (MInference ×0.45, FlexPrefill ×0.76). Figure 6a reports further speedup growth at 256K but gives no number in the text.

**Versus token eviction at matched cost** (Table 5, RULER, LLaMA-3.1-8B; attention speedup ≈ ×1.5 for all):

| Method | 4K | 8K | 16K | 32K | 64K | 128K | Avg | Attn speedup |
|---|---|---|---|---|---|---|---|---|
| FlashAttention | 95.82 | 92.77 | 91.02 | 84.87 | 83.43 | 74.15 | 87.01 | ×1.00 |
| PyramidInfer | 83.15 | 79.27 | 80.74 | 80.42 | 77.98 | 69.35 | 78.49 | ×1.49 |
| GemFilter | 92.50 | 90.87 | 88.78 | 85.01 | 80.42 | 73.15 | 85.12 | ×1.53 |
| FastKV | 94.25 | 91.30 | 89.78 | 84.57 | 81.54 | 72.39 | 85.64 | ×1.50 |
| Token Sparse (τ = 0.01) | 95.82 | 92.71 | 91.46 | 84.81 | 83.00 | 73.25 | **86.84** | ×1.51 |

Against OrthoRank (non-evicting but FFN-skipping), compared on *prefill* speedup (Table 13): OrthoRank p = 0.30 gives 79.36 at ×1.27 and p = 0.35 gives 78.38 at ×1.31; Token Sparse gives 87.02 at ×1.27 (τ = 0.005) and 86.84 at ×1.32 (τ = 0.010) — a ~17-point gap at 128K. Note this table also confirms the τ = 0.005 configuration's end-to-end prefill gain is ×1.27, the same as Table 10.

**Ablations that matter**

- *Scoring signal* (Table 7): random queries 84.95, query-pooling 86.43, recent queries 87.02 — the last_q proxy inherited from MInference/FastKV is the right one.
- *Which layers* (Table 8): sparsifying the lowest-drift third of layers gives 86.89 (LLaMA) / 67.15 (Mistral); the highest-drift third gives 72.13 / **25.55** (128K: 38.17 / 4.03). Drift is model-agnostic as a selector but the penalty for choosing wrong is severe.
- *How many layers* (Table 9, δ sweep, LLaMA): δ = 0.1 → 86.89 @ ×1.12; 0.3 → 86.92 @ ×1.24; **0.5 → 87.02 @ ×1.36**; 0.7 → 85.35 @ ×1.46; 0.9 → 79.16 @ ×1.53. Beyond half the layers the speedup gain is marginal and accuracy falls off a cliff (128K: 73.68 → 56.41).
- *Reversibility*: the paper attributes its edge over FastKV/GemFilter to the residual path keeping skipped tokens available and to per-head token sets; NIAH (Fig. 7) with FlexPrefill + Token Sparse scores higher than FlexPrefill alone.

**Where it breaks**

- *Short contexts*: no end-to-end gain at 8K on either GPU (×0.97 A100, ×0.86 A6000); the authors' own limitation section says gains "are naturally smaller at short context lengths".
- *Modest ceiling on its own*: ×1.36 attention / ×1.27 TTFT at 128K when applied to dense FlashAttention. The large numbers only appear stacked on a block-sparse kernel.
- *Over-sparsification*: δ > 0.5 or high-drift layers collapse long-context accuracy; the "within 1%" robustness claim (Fig. 5b) holds for the τ sweep at δ = 0.5, not for the layer-selection knobs.
- *Retrieval-heavy tasks when stacked*: InfiniteBench Retr.KV loses 2–4 points on top of MInference/FlexPrefill; the abstract's "consistently improves accuracy–latency trade-off" is true of the Pareto frontier, not of every cell.
- *Abstract vs. body*: the "up to ×3.23 attention speedup at 128K with less than 1% accuracy degradation" figure appears nowhere in the tables — the closest are ×2.76 (FlexPrefill + TS, 87.27, Table 1) and ×3.49 (X-Attention + TS, Table 12, but that stack sits 3.5 points below dense because X-Attention itself does). ×3.23 presumably comes from the τ sweep in Fig. 5b, which is only plotted.
- *Decode untouched*: TPOT and KV memory are unchanged by construction.
- *Scope*: text-only prefill; extension to decoding and vision-language models is listed as future work.

## Related

- [[sparse-attention]] — the family; this is the token-granularity, reversible, kernel-agnostic member (no concept page yet).
- [[minference-dynamic-sparse-attention-jiang-2024]] — block/column/diagonal sparsity from the same "recent queries × all keys" estimator; used here both as a baseline and as a substrate to compose with.
- [[claa-cross-layer-attention-aggregation-mcdanel-2026]] — the contrasting *token-pruning* approach (FastKV lineage, drops tokens from the sequence and the KV cache); the two papers share the FastKV authors' scoring signal but make opposite reversibility choices.
- [[flash-attention]] / [[flash-attention-2-dao-2023]] — the dense baseline; the compressed Q̂, K̂, V̂ are fed to it unmodified.
- [[attention-mechanism]] / [[multi-head-attention]] — per-head token selection is the key design point; heads in the same layer rank tokens differently.
- [[residual-stream]] / [[residual-connection]] — the mechanism that makes selection reversible: unselected tokens carry forward through the residual, not through attention.
- [[kv-cache]] — full cache retained; the paper pairs prefill sparsification with (unchanged) dense decode, and contrasts with SnapKV/H2O-style decode eviction.
- [[long-context-llm]] — the ≥128K regime where the tail of low-mass tokens is long enough to matter.
- [[llm-inference-roofline-yuan-2024]] — why attention-only methods saturate at the attention share of prefill (~70–80% at 128K here).
- [[methods-to-speed-up-prompt-processing]] — the wiki's synthesis; this belongs in the "cut the quadratic term" tier as a composable add-on.
- [[minimax-sparse-attention-lai-2026]] / [[flashmemory-lookahead-sparse-attention-wang-2026]] — trained sparse-attention designs; Token Sparse Attention is training-free and post-hoc.
- [[prompt-compression]] — the prompt-level (LLMLingua) alternative the related-work section separates from in-model token selection.
- [[llama]] — LLaMA-3.1-8B-Instruct is the primary evaluation model.

## References

- [arXiv:2602.03216](https://arxiv.org/abs/2602.03216) — v3, 29 May 2026; ICML 2026.
- Code: https://github.com/dongwonjo/Token-Sparse-Attention
