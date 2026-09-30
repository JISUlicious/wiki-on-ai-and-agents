---
title: "Native Sparse Attention: Hardware-Aligned and Natively Trainable Sparse Attention"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2502.11089"
authors:
  - Jingyang Yuan
  - Huazuo Gao
  - Damai Dai
  - Junyu Luo
  - Liang Zhao
  - Zhengyan Zhang
  - Zhenda Xie
  - Y. X. Wei
  - Lean Wang
  - Zhiping Xiao
  - Yuqing Wang
  - Chong Ruan
  - Ming Zhang
  - Wenfeng Liang
  - Wangding Zeng
year: 2025
venue: ACL 2025 (Best Paper Award)
status: complete
importance: high
tags:
  - 2025
  - paper
---

# Native Sparse Attention: Hardware-Aligned and Natively Trainable Sparse Attention

**Jingyang Yuan, Huazuo Gao, Damai Dai, Junyu Luo, Liang Zhao, Zhengyan Zhang, Zhenda Xie, Y. X. Wei, Lean Wang, Zhiping Xiao, Yuqing Wang, Chong Ruan, Ming Zhang, Wenfeng Liang, Wangding Zeng** ([[deepseek|DeepSeek-AI]]; [[peking-university|Peking University]], Key Laboratory for Multimedia Information Processing / PKU-Anker LLM Lab; [[university-of-washington|University of Washington]]. Yuan's contribution was made during a DeepSeek internship; corresponding authors Yuan and M. Zhang at PKU, Zeng and Liang at DeepSeek) — [arXiv:2502.11089](https://arxiv.org/abs/2502.11089), cs.CL, v1 16 Feb 2025, v2 27 Feb 2025 (the version read here). **Venue: ACL 2025, one of four Best Paper Awards** — confirmed on the ACL 2025 awards page; the arXiv record itself carries no venue, comments or journal-ref field. No code release is announced in the paper.

> [!important] What this paper is, in one paragraph
> A sparse attention *architecture* meant to be pretrained from scratch, not bolted onto a dense model. Each query attends through three parallel branches over the same KV history — **compressed** block summaries (a learned MLP squeezes every 32-token block, stride 16, into one key and one value), **selected** raw blocks (the top-16 64-token blocks, ranked by the compressed branch's own attention probabilities, summed across the 16 heads of a GQA group so the whole group loads one shared block set), and a **sliding window** of the last 512 tokens — and a per-query sigmoid **gate** mixes the three outputs. Trained this way on a 27B-total / 3B-active MoE for 260–270B tokens, it matches or beats full attention on general benchmarks (0.456 vs 0.443), LongBench (0.469 vs 0.437) and [[aime|AIME]] after R1-distilled SFT (0.146 vs 0.092 at 16k), while a Triton kernel that loads a GQA group's queries together delivers **9.0× forward / 6.0× backward** over Triton FlashAttention-2 at 64k, and decoding needs **11.6× fewer KV token-loads** at 64k — an *expected*, not measured, decode speedup. The thesis is that post-hoc sparsity loses accuracy because it pulls a model off its pretrained trajectory, whereas sparsity present during training does not — and the loss curve sits *below* full attention for the whole run.

## Problem

Two arguments, each its own section of the paper.

**"The illusion of efficient inference" (§2.1).** Sparse methods that keep a dense pretrained backbone rarely convert theoretical FLOP savings into latency. Two failure modes: *phase-restricted sparsity* — H2O sparsifies decode but needs a full attention-map pass at prefill, [[minference-dynamic-sparse-attention-jiang-2024|MInference]] sparsifies only prefill — so one stage always costs as much as full attention (bad for summarisation and code, which are prefill-heavy, and for long CoT, which is decode-heavy); and *incompatibility with [[grouped-query-attention|GQA]]/MQA* — Quest lets each head pick its own KV subset, so on a GQA model the bytes actually loaded are the *union* over the group, and memory traffic barely falls even though FLOPs do. The framing throughout is [[arithmetic-intensity]]: prefill and training are compute-bound, so the goal there is fewer FLOPs; decode is memory-bound, so the goal is fewer KV bytes per token. The paper cites a theoretical estimate that softmax attention is **70–80% of decode latency at 64k**.

**"The myth of trainable sparsity" (§2.2).** Applying sparsity after pretraining "forces models to deviate from their pretrained optimization trajectory": MagicPIG's measurement that the **top 20% of attention entries carry only ~70% of the mass** means retrieval heads get pruned at inference. And existing methods cannot simply be switched on during training: ClusterKV's k-means and MagicPIG's SimHash are non-differentiable and break gradient flow through selection; HashAttention's token-granular gathers defeat [[flash-attention]]-style contiguous block loads, so a trainable version runs at low hardware utilisation. Long-context *training* — pretraining on long documents, long-context SFT, RL over long chains of thought — is the workload the authors actually care about (the R1 era is invoked on page 1), and none of the inference-only methods help there.

## Mechanism

For query **q**ₜ the output is a gated sum over three remapped KV sets (Eq. 5):

> **o**ₜ = Σ_{c ∈ {cmp, slc, win}} gₜᶜ · Attn(**q**ₜ, K̃ₜᶜ, Ṽₜᶜ), with gₜᶜ ∈ [0, 1] produced by an MLP + sigmoid on the input features.

Each branch has **its own key/value projections** ("independent keys and values for three branches") — a deliberate choice to stop the branches shortcutting through one another, at "marginal" parameter cost. Sparsity means Nₜ = Σ|K̃ₜᶜ| ≪ t.

**1. Compression (coarse).** Blocks of length *l* = 32 with stride *d* = 16 (overlapping; *d* < *l* "to mitigate information fragmentation"); a learnable MLP φ with intra-block position encoding maps each block to one compressed key (and, separately, one value). That yields ⌊(t−l)/d⌋ + 1 compressed tokens — 2,047 at 32k, 4,095 at 64k — and is the branch that gives every query cheap global reach.

**2. Selection (fine).** Raw tokens are kept for the top-*n* = 16 selection blocks of size *l′* = 64 (1,024 tokens), with the first block and the two most recent blocks always on. The trick is *where the block scores come from*: not a separate scoring pass but the **softmax attention probabilities of the compression branch** (Eq. 8), remapped from the compression grid onto the selection grid by summing overlapping entries (Eq. 9). Then, for GQA/MQA, the per-head scores are **summed over the H = 16 query heads of a group** (Eq. 10) so all heads in the group select the same blocks and the decode-time KV load is one contiguous set per group rather than a union. Selection itself is a hard top-*n* (Eqs. 11–12: rank, take, concatenate), but everything upstream of it is the compressed branch's ordinary attention, so no auxiliary loss is needed — contrast the SeerAttention-style variant in § Alternatives.

**3. Sliding window (local).** The last *w* = 512 tokens. The motivation is about training dynamics, not accuracy: "local patterns typically adapt faster and can dominate the learning process", starving the other two branches; giving locality its own branch (and its own K/V) lets compression and selection learn long-range structure.

Blockwise selection is justified two ways: hardware (contiguous block reads and Tensor Core tiles, "as exemplified by FlashAttention's block-based design") and data (attention maps of the 27B full-attention model show block-clustered scores, Fig. 8 — the same spatial continuity MInference exploits).

### Kernel

Compression and window attention run on stock FlashAttention-2 kernels. The selection branch gets a **Triton** kernel (Fig. 3) whose single idea is to invert FlashAttention's loop order for sparse KV: instead of loading a temporally contiguous *query* block (whose queries would want disjoint KV blocks), load **all 16 heads of one GQA group at one position** into SRAM — they share the block index set 𝓘ₜ — then stream that group's selected contiguous KV blocks (kernel block size B_k with B_k | *l′*) through the inner loop, and let Triton's grid scheduler own the outer loop over positions because the inner-loop length (∝ *n*) is nearly constant across queries. The paper calls the result "near-optimal arithmetic intensity": no redundant KV transfers (group-wise sharing) and balanced SM load. A backward kernel exists (the 6.0× number) but is not described.

## Setup

| | |
|---|---|
| Backbone | GQA + [[mixture-of-experts\|MoE]] transformer, **27B total / 3B active** |
| Depth / width | 30 layers, hidden 2560 |
| Attention | 64 heads in **4 GQA groups** (16 heads per group), d_q = d_k = 192, d_v = 128 |
| MoE | DeepSeekMoE, 72 routed + 2 shared experts, top-6; layer 1 uses a SwiGLU MLP for stability |
| NSA hyperparameters | *l* = 32, *d* = 16, *l′* = 64, *n* = 16 (incl. 1 initial + 2 local), *w* = 512 |
| Pretraining | **270B tokens at 8k** (§4.1) — the abstract and introduction say **260B**; see § Claims |
| Long-context stage | continued training + SFT at 32k with YaRN |
| Baseline | the identical model with full attention, same data, both trained "to full convergence" |
| Efficiency hardware | 8× A100; Triton NSA vs **Triton** FlashAttention-2 |

*Derived, and it checks against Table 4:* per query NSA loads t/16 + 1,536 tokens (compressed + 16·64 selected + 512 window): 2,048 at 8k, 5,632 at 64k. Averaged over positions in a 32k sequence that is 1,024 + 1,536 = **2,560**, exactly the budget the paper hands the sparse baselines on LongBench. Fig. 4: NSA's pretraining loss is below full attention's for the whole run, both smooth.

## Results

### General benchmarks (Table 1, pretrained base)

| | MMLU | MMLU-Pro | CMMLU | BBH | GSM8K | MATH | DROP | MBPP | HumanEval | **Avg** |
|---|---|---|---|---|---|---|---|---|---|---|
| Full attention | **0.567** | 0.279 | 0.576 | 0.497 | 0.486 | 0.263 | 0.503 | **0.482** | 0.335 | 0.443 |
| NSA | 0.565 | **0.286** | **0.587** | **0.521** | **0.520** | **0.264** | **0.545** | 0.466 | **0.348** | **0.456** |

7 of 9 to NSA (checks); it loses [[mmlu|MMLU]] by 0.002 and MBPP by 0.016. The biggest wins are reasoning-flavoured: DROP +0.042, GSM8K +0.034 (both check). The authors' explanation — sparse pretraining "forces model to focus on the most important information" — is a hypothesis, not tested.

### LongBench (Table 2; every sparse baseline at a 2,560-token budget)

| | MFQA-en | MFQA-zh | Qasper | HPQ | 2Wiki | GovRpt | Dur | PassR-en | PassR-zh | LCC | **Avg** |
|---|---|---|---|---|---|---|---|---|---|---|---|
| H2O | 0.428 | 0.429 | 0.308 | 0.112 | 0.101 | 0.231 | 0.208 | 0.704 | 0.421 | 0.092 | 0.303 |
| InfLLM | 0.474 | 0.517 | 0.356 | 0.306 | 0.250 | 0.277 | 0.257 | 0.766 | 0.486 | 0.143 | 0.383 |
| Quest | 0.495 | 0.561 | 0.365 | 0.295 | 0.245 | 0.293 | 0.257 | 0.792 | 0.478 | 0.135 | 0.392 |
| Exact-Top | 0.502 | 0.605 | 0.397 | 0.321 | 0.288 | 0.316 | 0.291 | 0.810 | 0.548 | 0.156 | 0.423 |
| Full attention | **0.512** | 0.623 | 0.409 | 0.350 | 0.305 | **0.324** | 0.294 | 0.830 | **0.560** | 0.163 | 0.437 |
| NSA | 0.503 | **0.624** | **0.432** | **0.437** | **0.356** | 0.307 | **0.341** | **0.905** | 0.550 | **0.232** | **0.469** |

The 2,560 activated tokens per query (128 leading + 512 local, StreamingLLM-style) is "the average number of tokens activated in NSA when handling 32k sequence lengths", so the comparison is sparsity-matched. NSA beats full attention by +0.032 and Exact-Top by +0.046 — both check — with the largest gaps on multi-hop QA (HPQ +0.087, 2Wiki +0.051), code (LCC +0.069) and passage retrieval (PassR-en +0.075). Low-scoring [[longbench|LongBench]] subsets were dropped "due to their low scores across all models". 64k needle-in-a-haystack: perfect at every depth (Fig. 5).

> [!note] Exact-Top is the interesting baseline
> Exact-Top computes the *full* attention matrix and keeps the exact top-*n* keys per query — the best any post-hoc selector could do at this budget on this dense model. It still trails full attention by 0.014 and NSA by 0.046. That gap is the paper's cleanest evidence for its thesis: at a matched budget the dense model *cannot* be sparsified without loss, while the natively sparse model has nothing to lose because it was never dense.

### Chain-of-thought after SFT (Table 3)

Both models are SFT'd on **10B tokens of 32k-length math traces distilled from [[deepseek-r1|DeepSeek-R1]]** (RL was judged ineffective at this scale) and scored on AIME 24, 16 samples per question at T = 0.7, top-p 0.95:

| Generation limit | Full Attention-R | NSA-R |
|---|---|---|
| 8,192 | 0.046 | **0.121** |
| 16,384 | 0.092 | **0.146** |

+0.075 and +0.054 (check). Sparse baselines are absent here because "sparse attention baselines do not support training" — which is the point of the paper.

### Speed (Fig. 6, Table 4; A100, Triton vs Triton FA-2)

| Context | Forward speedup | Backward speedup | Decode KV loaded (token-equivalents), full vs NSA | Expected decode speedup |
|---|---|---|---|---|
| 8k | 2.1× | 1.1× | 8,192 vs 2,048 | 4.0× |
| 16k | 3.8× | 2.0× | 16,384 vs 2,560 | 6.4× |
| 32k | 6.3× | 3.4× | 32,768 vs 3,584 | 9.1× |
| 64k | **9.0×** | **6.0×** | 65,536 vs 5,632 | **11.6×** |

Forward/backward are Fig. 6's labels (at 64k: FA-2 ≈ 1,000 ms → NSA ≈ 110 ms forward; ≈ 2,600 ms → ≈ 430 ms backward, with the sparse-selection kernel and the compression + window kernels each roughly half of NSA's time). The **decode column is arithmetic, not a measurement**: per step NSA loads at most ⌊(s−l)/d⌋ compressed + *n·l′* selected + *w* window tokens, and the paper says that because decode is memory-bound "the expected speedup is approximately linear with the volume of memory access". Fig. 1's "11.6×" bar is this number. There is **no measured decode latency, no end-to-end prefill/TTFT and no throughput figure anywhere in the paper**: "9×" is the attention kernel, and attention is only part of a layer — see the attention-share rule in [[sparse-attention]].

## Alternatives the authors tried first (§6.1)

On a 3B model of the same design, three trainable-sparse alternatives were compared with NSA and full attention by loss curve (Fig. 7 — the text mistakenly points to Fig. 8, the attention-map figure):

- **Key clustering (ClusterKV-style)** — rejected for dynamic-clustering overhead, inter-cluster load imbalance that compounds MoE expert-parallel imbalance, and mandatory periodic re-clustering with chunk-sequential training.
- **Auxiliary-loss block selection (SeerAttention-style)** — extra per-token queries and per-block representative keys, supervised by KL against mean-pooled block attention; "increases operator overhead and often degrades model performance".
- **Parameter-free heuristic (Quest-style)** — query · coordinate-wise min/max of the key chunk, with and without a 1,000-step full-attention cold start; "low recall rates".

Both trainable alternatives "exhibited inferior loss". This is the closest the paper comes to justifying the three-branch design; there is no ablation of the branches themselves, of *n*, *l*, *w*, or of the gate.

## Claims that did not check out, or need qualification

- **260B vs 270B pretraining tokens.** The abstract and §1 say "260B tokens"; §4.1 says "270B tokens of 8k-length texts". Both readings are present in the v2 PDF and HTML. Unreconciled.
- **11.6× decode is expected, not measured** (Table 4 is memory-access volume in token-equivalents; no decode timing exists). 9.0× / 6.0× are attention-kernel timings against a *Triton* FA-2 port, not the CUDA FA-2/FA-3 kernels people deploy; at 8k the backward kernel is only 1.1× faster.
- **"Surpasses full attention"** holds on averages (0.456 vs 0.443; 0.469 vs 0.437) but not uniformly (loses MMLU, MBPP, MFQA-en, GovRpt, PassR-zh), and every baseline was trained by the authors — there is no external checkpoint or third-party reproduction in the paper.
- **Fig. 8 misreference** in §6.1 (should be Fig. 7).
- **Wall-clock training savings are not reported**: the only training-cost statement is the kernel speedup; total pretraining time for NSA vs full attention is absent even though "reducing pretraining computation" is in the abstract.
- Everything else verified against the body: 27B/3B, 30 layers, 2560, 64 heads / 4 groups, 192/128, 72+2 experts top-6, *l*/*d*/*l′*/*n*/*w* = 32/16/64/16/512, the 2,560-token baseline budget, all Table 1–3 deltas, Table 4 ratios (8192/2048 = 4.0, 16384/2560 = 6.4, 32768/3584 = 9.14, 65536/5632 = 11.64), ACL 2025 Best Paper.

## Limitations

- **One scale, one family, one lab's checkpoints.** A 27B/3B MoE (plus a 3B for §6.1), all pretrained by the authors; no result on adapting an *existing* dense model, which is exactly what [[moba-mixture-of-block-attention-lu-2025|MoBA]] and [[deepseek-v3.2-sparse-attention-deepseek-2025|DSA]] do. Whether NSA's edge survives at frontier scale is answered only indirectly, by DeepSeek's later choice of a simpler indexer-plus-top-k design for V3.2.
- **New parameters, new architecture.** Three sets of K/V projections, the compression MLP φ and the gate MLP — NSA cannot be switched on inside a dense checkpoint; it has to be pretrained.
- **Longest evaluated context is 64k** (NIAH) after 32k training; no 128k+ or million-token result.
- **Kernel numbers are A100 + Triton only**; no H100/H800, no vendor FlashAttention comparison.
- **Fixed budgets** (*n* = 16 blocks, *w* = 512): the activated set grows only through the compression branch (t/16), so at long contexts compression dominates the per-query cost and the sparsity ratio asymptotes to **~16× fewer tokens than full attention** — 11.6× at 64k is most of the way there. The paper does not discuss this ceiling.
- **No ablations** of the three branches, the per-branch K/V choice, or the gate.

## Related

- [[trained-sparse-attention]] — the concept page this source anchors; NSA is the from-scratch end of the spectrum.
- [[moba-mixture-of-block-attention-lu-2025]] — Moonshot's design from two days later: parameter-free mean-pooled-key gate, per-head top-k, no compression or window branches, switchable into an existing dense model (Llama 3.1 8B). The two papers are the natural pair.
- [[deepseek-v3.2-sparse-attention-deepseek-2025]] — DeepSeek's production successor: a learned FP8 "lightning indexer" + top-2048 selection under MLA, adapted into V3.1-Terminus by continued training. It cites this paper for the rule that KV entries must be shared across query heads at the kernel level.
- [[sparse-attention]] · [[sparse-prefill]] — the training-free lineage this paper argues against; [[minference-dynamic-sparse-attention-jiang-2024]] is its named example of prefill-only sparsity. The attention-share rule there applies to NSA's 9×: it is a kernel speedup.
- [[minimax-sparse-attention-lai-2026]] · [[qwen3.8-flash-next]] — later trained-sparse designs; both keep NSA's *group-shared blockwise selection* and drop its compression and window branches.
- [[flash-attention]] — the kernel substrate; NSA's contribution is a group-centric loop order for sparse KV.
- [[grouped-query-attention]] · [[kv-cache]] · [[arithmetic-intensity]] · [[llm-inference-roofline-yuan-2024]] — why selection is summed across a GQA group: decode is bytes-bound and a per-head union defeats the point.
- [[mixture-of-experts]] — the backbone is DeepSeekMoE; MoE load imbalance is also why key clustering was rejected.
- [[deepseek-r1]] · [[deepseek-r1-2025]] — the SFT teacher for the AIME experiment, and the "long CoT is decode-heavy" motivation.
- [[longbench]] · [[mmlu]] · [[humaneval]] · [[attention-sink]] — the leading-128 + local-512 baseline budget is StreamingLLM's sink + window.
- [[long-context-llm]] · [[methods-to-speed-up-prompt-processing]] · [[deepseek]].
- [[multi-head-latent-attention]] — NSA is instantiated on GQA, DSA on MLA, and the difference matters for the kernel.

## References

- [arXiv:2502.11089](https://arxiv.org/abs/2502.11089) · ACL 2025 Best Paper ([awards page](https://2025.aclweb.org/program/awards/))
- MInference: Jiang et al., [arXiv:2407.02490](https://arxiv.org/abs/2407.02490) — the prefill-only foil ([[minference-dynamic-sparse-attention-jiang-2024]]).
- Quest: Tang et al., ICML 2024 — per-head blockwise selection; source of the GQA-union argument.
- H2O: Zhang et al., NeurIPS 2023 — decode-only eviction behind a full prefill.
- MagicPIG: Chen et al., [arXiv:2410.16179](https://arxiv.org/abs/2410.16179) — the "top 20% covers 70%" figure.
- SeerAttention: Gao et al., [arXiv:2410.13276](https://arxiv.org/abs/2410.13276) — the auxiliary-loss selector NSA compares against in §6.1.
- InfLLM: Xiao et al., 2024; ClusterKV: Liu et al., 2024; HashAttention: Desai et al., 2024; StreamingLLM: Xiao et al., 2023.
- FlashAttention: Dao et al., 2022; Triton: Tillet et al., 2019; GQA: Ainslie et al., [arXiv:2305.13245](https://arxiv.org/abs/2305.13245); YaRN: Peng et al., 2024; DeepSeekMoE: Dai et al., 2024.
