---
title: "Cross-Model KV Cache Transfer in LLM Families: A Closed-Form Linear Mapping for Prefill Reuse"
type: source
created: 2026-08-11
updated: 2026-08-11
sources:
  - cross-model-kv-cache-transfer-heo-2026.md
arxiv_id: "2608.03893"
year: 2026
authors:
  - Taekyung Heo
  - Rasoul Shafipour
  - Ritchie Zhao
  - Maximilian Golub
  - Bita Darvish Rouhani
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Cross-Model KV Cache Transfer in LLM Families: A Closed-Form Linear Mapping for Prefill Reuse

**Taekyung Heo, Rasoul Shafipour, Ritchie Zhao, Maximilian Golub, Mohammad Mahdi Kamani, Ritika Borkar, Makesh Tarun Chandran, Pantea Zardoshti, Bita Darvish Rouhani** (NVIDIA) — [arXiv:2608.03893](https://arxiv.org/abs/2608.03893), cs.LG, 4 Aug 2026. Marked "Preprint"; no venue given.

## Summary

Production serving increasingly swaps between different-sized members of the *same* model family — cost-quality cascading, mid-conversation switching, routing — and every swap forces the receiving model to **repay the prefill from scratch**. [[prefix-caching]] does not help, because it reuses [[kv-cache]] entries only within a single model. Since prefill's only output *is* the KV cache, the authors reframe the swap as a representation problem: transform the source model's cache into the format the target expects. They call this **cross-model KV cache transfer** and restrict the study to within-family, bidirectional transfer (small→large upgrades quality, large→small cuts cost).

The central empirical finding is that this cross-model relationship is **largely linear**. Probing with per-token OLS on Qwen3 14B→32B, a *single* source layer explains **56% of the variance in the target's keys and 32% in values**, rising to **79% and 65%** when multiple source layers are used; the best single source-target cell reaches R²=0.81 for RoPE-stripped keys. Keys are consistently more predictable than values (a ~0.2 R² gap), and stripping RoPE sharpens the layer-to-layer diagonal. This licenses a **closed-form, gradient-free mapper**: per (target layer, head), select the top-k most predictive source layers and concatenate their KV, map keys in RoPE-stripped *content space*, and solve ridge regression (λ=0.01) on a small calibration set of **500 FineWeb-Edu sequences of 1,024 tokens** (stride-4 subsampled to ≈128K tokens per target head). Fitting takes ~47-87 min per pair on one 8×H100 node with no backpropagation.

Results are honest about a split outcome. Across **six matched-KV pairs in three families** (Qwen3, Llama 3.1, Ministral 3 — pairs sharing KV head count and per-head dimension), **four pairs retain 73-98%** of the receiver's standalone accuracy averaged over five benchmarks, while **two Ministral 3 pairs collapse to 42-44%** (11-15% floor-normalized). The diagnosis is that *where* the residual error lands matters more than its magnitude: calibration R² does not predict retention (r=−0.20), but attention-output cosine does (r=+0.57 over 12 pair evaluations). A drop-in nonlinear MLP recovers the failures (+24.3 to +36.8 pp HellaSwag, putting all four tested pairs above 90%) while very slightly *hurting* the pairs ridge already handles.

## Key points

**Linear structure (Qwen3 14B→32B)** — all three of the headline numbers verify exactly:

| Probe | Keys | Values |
|---|---|---|
| Single source layer (R², head-averaged) | 56% | 32% |
| Multiple source layers | 79% | 65% |
| Best single source→target cell (K_stripped) | R²=0.81 | — |

k=1 captures only 66% of the k=all R² for stripped keys and 42% for values; most of the gain comes between k=1 and k=4, saturating by k≈6.

**Main results — retention vs. the target's own standalone prefill** (ridge, content-space, at each pair's selected k; Avg_fn is floor-normalized):

| Family | Pair (k) | Avg | Avg_fn | ARC-C | HellaSwag | WinoGrande | MMLU | GSM8K |
|---|---|---|---|---|---|---|---|---|
| Qwen3 | 14B→32B (8) | **97.6%** | 96.3% | 101.0% | 97.6% | 98.5% | 95.0% | 95.6% |
| Qwen3 | 8B→32B (12) | 87.5% | 80.7% | 94.0% | 95.2% | 91.0% | 88.5% | 68.8% |
| Llama 3.1 | 8B→70B (20) | 72.8% | 62.9% | 90.9% | 94.4% | 87.1% | 73.3% | 18.2% |
| Ministral 3 | 3B→8B (all) | 76.2% | 65.9% | 90.6% | 93.3% | 91.3% | 69.4% | 36.6% |
| Ministral 3 | 3B→14B (20) | **44.2%** | 14.7% | 43.6% | 68.0% | 74.0% | 32.0% | 3.2% |
| Ministral 3 | 8B→14B (12) | **41.6%** | 11.1% | 40.7% | 58.7% | 74.2% | 32.7% | 1.6% |

The two failures are both **Ministral 3 → 14B**. Llama 3.1 8B→70B survives despite being the most extreme parameter ratio evaluated — so scale gap is not the discriminator. Note GSM8K is the most fragile benchmark everywhere (18.2% even on a Tier-1 pair).

**Prefill latency vs. re-prefill (Qwen3 14B↔32B)** — the paper *does* report end-to-end numbers:

| Seq len | S→L mapper | S→L re-prefill | Speedup | L→S mapper | L→S re-prefill | Speedup |
|---|---|---|---|---|---|---|
| 64 | 14.0 ms | 61.7 ms | 4× | 11.6 ms | 39.2 ms | 3× |
| 8K | 67.8 ms | 1154.8 ms | 17× | 101.9 ms | 501.0 ms | 5× |
| 32K | 277.6 ms | 6975.3 ms | 25× | 427.1 ms | 2952.7 ms | 7× |

Headline range is **2.7-25×** (the 2.7× low end comes from the full sweep in Appendix G; the main text quotes 4-25× S→L and 3-7× L→S). Speedup grows with sequence length because the mapper is a per-layer batched matmul replacing the whole transformer body, so its wall time scales far better than re-prefill.

**Ridge vs. MLP (HellaSwag retention)** — two 1,024-unit ReLU hidden layers per (layer, head, K|V), same MSE objective:

| Pair | Ridge | MLP | Δ |
|---|---|---|---|
| Qwen3 14B→32B | 97.6% | 97.3% | −0.3 pp |
| Ministral 3B→8B | 93.3% | 91.8% | −1.5 pp |
| Ministral 3B→14B | 68.0% | 92.3% | **+24.3 pp** |
| Ministral 8B→14B | 58.7% | 95.5% | **+36.8 pp** |

The MLP lowers K error concentration (Δ≈−2.5) and raises attention-output cosine (Δ≈+0.45) — it redistributes error into attention-*irrelevant* directions rather than shrinking it. On failure pairs ridge's eval-domain R²_K is deeply negative (the calibration fit does not extrapolate to HellaSwag tokens); the MLP closes most, not all, of that gap.

**Ablation (Qwen3 14B→32B)** — cross-layer source selection is the single largest contributor: dropping k from 8 to 1 cuts K R² from 0.79 to 0.56. Disabling inference-time RoPE handling collapses MMLU (78.09→25.79) and GSM8K (90.98→4.17) to near-chance while HellaSwag falls only ~5 pp. Fully RoPE-coupled fitting is within noise on *short* contexts — the content-space formulation matters because it extends to 32k by construction. Calibration is robust: flat across four orders of magnitude in λ (collapse only at λ=1), flat past N=200 sequences (N=50 within ~1.6 pp), but calibration *domain* costs real accuracy (CodeAlpaca −5.24 pp on HellaSwag; Wikipedia within noise).

**Multi-turn stability** — CoQA, 100 conversations of ~15 turns across five domains, Qwen3 14B↔32B. Drift stays small both ways: the S→L gap widens **1.7 pp from turn 1 to turn 10** (the mapper holds steady while the 32B ceiling rises), and L→S drift grows linearly at **0.33 pp/turn**. No cascading failure within ten turns, though the authors note linear L→S drift would accumulate over very long sessions.

**Mapper cost** — 1.01-3.36 B parameters, 4-12 GB storage per pair. This is a real deployment tax: you store a mapper per directed pair.

**Stated limitations** — (1) single-domain calibration (FineWeb-Edu only; the substitution sweep does not bound narrow domains like medicine or law); (2) k is selected on the same log-likelihood benchmarks reported (effect bounded at 2.49 pp, but not out-of-sample); (3) matched-KV is empirical — all six pairs are matched by construction, mismatched-KV is untested; (4) scope is within-family, dense full-attention only. Cross-family transfer (e.g. Qwen3→Llama 3.1) is explicitly open.

## Concepts & entities

- [[cross-model-kv-cache-transfer]] — the paper's own framing: reusing one model's prefilled cache in another model, generalizing intra-model reuse.
- [[kv-cache]] — the object being transferred; prefill's output and therefore the whole target of the reuse.
- [[prefix-caching]] — the incumbent mitigation, explicitly limited to reuse within a single model.
- [[grouped-query-attention]] — all pairs use GQA; "matched-KV" is defined by equal KV head count and per-head dimension.
- [[transformer-architecture]] — the mapper replaces the target's transformer body with a per-layer batched matmul.
- [[attention-mechanism]] — retention is governed by attention-output cosine, not reconstruction R².
- [[long-context-llm]] — RoPE-stripped content-space mapping is what makes the fit valid out to 32k tokens.
- [[learned-orchestration]] — the motivating deployment pattern: cascading, routing, mid-conversation model switching.
- [[methods-to-speed-up-prompt-processing]] — cross-model transfer as a prefill-avoidance technique.
- [[droidspeak-kv-cache-sharing-liu-2024]] — closest prior work; cited as requiring identical architecture.
- [[knowledge-distillation]] — contrast: this transfers activations at serve time rather than training a student.

## References

- [arXiv:2608.03893](https://arxiv.org/abs/2608.03893)
