---
title: "Kimi Linear: An Expressive, Efficient Attention Architecture"
type: source
created: 2026-07-28
updated: 2026-07-28
sources:
  - kimi-linear-kimi-team-2025.md
arxiv_id: "2510.26692"
year: 2025
authors:
  - Kimi Team
  - Yu Zhang
  - Zongyu Lin
  - Xingcheng Yao
  - Jiaxi Hu
introduces:
  - [[linear-attention]]
tags:
  - 2025
  - paper
status: complete
importance: high
---

# Kimi Linear: An Expressive, Efficient Attention Architecture

**Kimi Team — Yu Zhang, Zongyu Lin, Xingcheng Yao, Jiaxi Hu, et al.** (Moonshot AI) — [arXiv:2510.26692](https://arxiv.org/abs/2510.26692).

## Summary

Kimi Linear is a hybrid linear-attention architecture that the authors claim is the first to *outperform* full attention under matched-scale, matched-recipe comparisons across short-context, long-context, and RL post-training regimes. Its core operator is **Kimi Delta Attention (KDA)**, a gated linear attention module that extends **Gated DeltaNet** by replacing GDN's coarse scalar/head-wise forget gate with a **channel-wise (per-feature-dimension) decay** `Diag(αt)`, in the spirit of Gated Linear Attention. The recurrence is `S_t = (I − β_t k_t k_tᵀ) Diag(α_t) S_{t−1} + β_t k_t v_tᵀ`, `o_t = S_tᵀ q_t` — i.e. fine-grained multiplicative decay followed by a DeltaNet-style Householder (delta-rule) correction, so each feature channel gets its own forgetting rate and the finite-state RNN memory is regulated more precisely.

KDA is a *constrained* Diagonal-Plus-Low-Rank (DPLR) transition: writing `S_t = (D − a_t b_tᵀ)S_{t−1} + k_t v_tᵀ` with `D = Diag(α_t)`, `a_t = β_t k_t`, `b_t = k_t ⊙ α_t`, both low-rank factors are bound to `k`. That binding removes two secondary-chunking steps needed for numerical stability under fine-grained decay and eliminates roughly three matrix multiplications in inter-chunk and output computation, so the bespoke chunkwise kernel runs at ~2× the speed of a general DPLR kernel for sequence lengths up to 64k while staying closer to the classical delta rule. Kernels ship in `flash-linear-attention` (`fla/ops/kda`) with a vLLM integration.

The model is **not pure linear attention**: KDA layers are interleaved with full **Multi-Head Latent Attention (MLA)** layers at a **uniform 3:1 layerwise ratio** (3 KDA layers per 1 MLA layer), a layer-level rather than head-level hybrid chosen for infra simplicity and training stability. All full-attention layers use **NoPE (no position encoding)**, delegating all positional/recency signal to KDA. The flagship configuration is a Moonlight-style MoE with **48B total / 3B activated parameters** (8 of 256 experts incl. one shared expert, sparsity 32, first layer dense); comparison runs use 1.4T tokens at 4k context with MuonClip + WSD, and the released checkpoint (`Kimi-Linear-48B-A3B-Instruct`) is trained on 5.7T tokens with up to 1M context. Headline efficiency: **up to 75% KV-cache reduction** and **up to ~6× decode throughput at 1M context**.

## Key points

**KDA vs. prior gated linear attention**

| Mechanism | State update | Gate granularity |
|---|---|---|
| DeltaNet | `(I − β_t k_t k_tᵀ)S_{t−1} + β_t k_t v_tᵀ` | none |
| Gated DeltaNet (GDN) | `α_t (I − β_t k_t k_tᵀ)S_{t−1} + β_t k_t v_tᵀ` | scalar / head-wise |
| Mamba2 | `α_t S_{t−1} + β_t k_t v_tᵀ` (no delta rule) | scalar |
| **KDA (this paper)** | `(I − β_t k_t k_tᵀ) Diag(α_t) S_{t−1} + β_t k_t v_tᵀ` | **channel-wise `α_t ∈ [0,1]^{d_k}`** |

- Neural parameterization: ShortConv (lightweight depthwise conv, small kernel e.g. 4) + Swish on q/k/v, L2Norm on q/k, `d_k = d_v = 128`, chunk size C = 64; `α_t` from a low-rank projection (rank = head dim); `β_t = Sigmoid(...)`; low-rank **Sigmoid** output gate + head-wise RMSNorm (also credited with alleviating attention sink).
- Ablation (16-layer scaling-law model, train/val PPL, lower better) — hybrid ratio **3:1 = 9.23 / 5.65** (best) vs 1:1 = 9.29 / 5.66, 7:1 = 9.23 / 5.70, 15:1 = 9.34 / 5.82, pure full attention 0:1 = 9.45 / 5.77. Component ablations: w/o output gate 9.25 / 5.67, Swish instead of Sigmoid gate 9.43 / 5.81, w/o conv layer 9.29 / 5.70.
- Synthetic tasks (palindrome, MQAR, LIFO stack tracking, 256→2048 tokens): KDA > GDN > Mamba2; Mamba2 (decay only, no delta rule) fails all three at this scale.
- Scaling law (5 MoE sizes, 653M–1.7B activated params, Chinchilla method): fitted `Kimi Linear: 2.2879·C^−0.0527` vs `MLA: 2.3092·C^−0.0536` → **~1.16× compute efficiency** over the MLA baseline at compute-optimal training.

**Efficiency (48B setting, all measured against full-attention MLA)**

| Metric | Number |
|---|---|
| KV cache | up to **75% reduction** (only 1 layer in 4 keeps a KV cache) |
| Decode TPOT @1M (Fig. 1b, larger batch enabled by smaller cache) | **1.84 ms vs 11.48 ms = 6.3×** |
| Decode TPOT @512k / @1M, batch size 1 (Fig. 7b) | **1.8× / 2.2×** |
| Prefill latency @512k / @1M (Fig. 7a) | **2.3× / 2.9×** |
| Prefill @4k–16k | comparable to MLA; gap opens from 128k onward |
| KDA vs GDN-H prefill | "virtually indistinguishable" — fine-grained decay adds negligible overhead |
| RULER 128k operating point (Fig. 1a) | 84.3 score at **3.98× acceleration** (Pareto-optimal) |
| KDA kernel vs general DPLR kernel | **~2×** up to 64k input length |

**Quality (1.4T-token matched runs; MLA = full attention, GDN-H = hybrid Gated DeltaNet)**

| Benchmark | MLA | GDN-H | Kimi Linear |
|---|---|---|---|
| MMLU (base) | 71.6 | 72.2 | **73.8** |
| MMLU-Pro (base) | 47.2 | 47.9 | **51.0** |
| BBH (base) | 71.6 | 70.6 | **72.9** |
| GSM8K (base) | 83.7 | 81.7 | **83.9** |
| EvalPlus (base) | 59.5 | **63.1** | 60.2 |
| MMLU-Pro (SFT) | 65.7 | 64.8 | **67.4** |
| GPQA-Diamond (SFT, Avg@8) | 57.1 | 58.6 | **62.1** |
| AIME 2025 (SFT, Avg@64) | 20.6 | 21.1 | **21.3** |
| LiveCodeBench v6 (SFT) | 25.1 | 25.4 | **26.0** |
| LiveBench (SFT) | 45.7 | **46.4** | 45.2 |
| MATH500 (SFT) | 80.8 | **83.0** | 81.2 |

- Long context @128k (Table 5): RULER **84.3** (MLA 81.3, GDN-H 80.5, Kimi Linear w/ RoPE 78.8); MRCR **29.6** (22.6 / 23.9 / 22.0); HELMET-ICL **90.0**; RepoQA **68.5** (63.0 / 63.0 / 66.5); average **54.5** vs 52.2 / 51.2 / 51.8. Two losses: LongBench v2 35.0 (MLA 36.1) and Frames 58.8 (MLA 60.5).
- Released 5.7T-token checkpoint: RULER **95.4 @128k** and **94.8 @1M**; beats Moonlight (16B/3B, same 5.7T tokens) nearly across the board (e.g. MMLU-Pro base 54.8 vs 42.4).
- RL (RLVR on in-house math set, identical algorithm/hyperparameters): Kimi Linear's training accuracy grows faster than MLA's and the gap widens; same pattern on MATH500 and AIME 2025 test curves.
- Overall ordering: short-context/SFT Kimi Linear > GDN-H > MLA; long-context Kimi Linear > MLA > GDN-H (GDN-H degrades at length).

**NoPE**

- NoPE is applied to **all** full-attention (MLA) layers; KDA is positioned as the model's primary position-aware operator, playing the role short convolutions or sliding-window attention play elsewhere.
- Justification: the gated delta rule is itself a *learnable, data-dependent multiplicative positional encoding* (Eq. 12), relaxing RoPE's orthogonality constraint and its fixed-frequency extrapolation failure modes.
- Ablation: Kimi Linear (RoPE) matches on short context but is clearly worse on long context (RULER 78.8 vs 84.3; avg 51.8 vs 54.5). The paper's explanation: with RoPE, the global layer carries a strong explicit short-range positional bias, overweighting local order and hurting mid-training context extension; NoPE spreads positional bias more evenly across depth.
- Practical bonuses: NoPE lets MLA layers collapse to pure **MQA** at inference, and removes RoPE base-frequency tuning / YaRN from long-context training.

**Stated limitations and caveats**

- Pure linear attention remains fundamentally limited by finite-state capacity for exact recall and copying — this is why full-attention layers are kept at all; long-context retrieval is called the primary bottleneck.
- Linear attention is still "limited by current hardware implementations and the absence of optimized inference infrastructure"; the paper's answer is shipping kernels + vLLM support, not a claim that the gap is closed.
- Higher hybrid ratios (7:1, 15:1) hurt validation loss; lower (1:1) costs inference efficiency — 3:1 is an empirical sweet spot, not a derived one.
- Scaling-law hyperparameters were tuned for MLA and reused verbatim for Kimi Linear; the authors expect a better curve than 1.16× with dedicated tuning.
- Loses to baselines on a few benchmarks (EvalPlus, MATH500, LiveBench, LongBench v2, Frames).
- Sparse attention retrieves fine-grained history better than linear attention (at the cost of storing the full KV cache); combining sparse and linear attention is left as future work.

## Concepts & entities

- [[linear-attention]] — the family KDA belongs to; the paper's whole argument is that gating + delta rule finally make it competitive with softmax attention.
- [[flash-kda-moonshot-2026]] — the open-sourced chunkwise KDA kernel (in `flash-linear-attention`, `fla/ops/kda`), ~2× faster than a general DPLR kernel, with vLLM integration.
- [[flash-kda-moonshot-2026]] — the later standalone kernel release that productionizes the KDA operator introduced here.
- [[moonshot-ai]] — the lab behind Kimi Linear, Moonlight, and the Kimi K1.5/K2 models the training recipe is inherited from.
- [[attention-mechanism]] — Table 6 unifies softmax attention, RoPE attention, linear attention, Mamba2, GLA, DeltaNet, GDN, RWKV-7, and KDA in a single recurrent/parallel formulation.
- [[transformer-architecture]] — Kimi Linear is a drop-in token-mixer replacement inside an otherwise standard MoE transformer backbone (Moonlight lineage).
- [[kv-cache]] — the 3:1 hybrid keeps a KV cache in only one layer of four, the source of the 75% memory reduction and the larger batch sizes behind the 6.3× decode speedup.
- [[long-context-llm]] — evaluated at 128k (RULER, MRCR, RepoQA, LongBench v2) and up to 1M context for the 5.7T checkpoint.
- [[mixture-of-experts]] — 48B total / 3B activated, 8 of 256 experts including one shared expert, first layer dense.
- [[state-space-model]] — Mamba2 is the SSM baseline KDA is measured against; the paper argues decay alone without a delta rule is insufficient.
- [[grouped-query-attention]] — NoPE lets the MLA layers be served as pure MQA (the degenerate one-group case) at inference.
- [[flash-attention]] — referenced as the kernel substrate for competing sparse-attention methods (e.g. MoBA's LSE-based chunk selection).
- [[attention-residuals-kimi-team-2026]] — later Moonshot AI work that replaces PreNorm residual accumulation with attention over layer outputs and is evaluated *on top of* the Kimi Linear 48B/3B architecture.
- [[minimax-sparse-attention-lai-2026]] — the sparse-attention branch of efficient long-context modeling that §7.1 contrasts with linear attention; the paper suggests hybridizing the two as future work.
- [[a-hippocampus-for-linear-attention-cui-2026]] — related follow-on line of work on compensating for what a finite recurrent state forgets.

## References

- [arXiv:2510.26692](https://arxiv.org/abs/2510.26692)
- Code: https://github.com/MoonshotAI/Kimi-Linear
- KDA kernels: https://github.com/fla-org/flash-linear-attention/tree/main/fla/ops/kda
- Checkpoint: https://huggingface.co/moonshotai/Kimi-Linear-48B-A3B-Instruct
