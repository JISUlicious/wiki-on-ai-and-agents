---
title: "MInference 1.0: Accelerating Pre-filling for Long-Context LLMs via Dynamic Sparse Attention"
type: source
created: 2026-09-03
updated: 2026-09-03
sources:
  - minference-dynamic-sparse-attention-jiang-2024.md
arxiv_id: "2407.02490"
venue: NeurIPS 2024 (Spotlight)
year: 2024
authors:
  - Huiqiang Jiang
  - Yucheng Li
  - Chengruidong Zhang
  - Qianhui Wu
  - Xufang Luo
  - Surin Ahn
  - Zhenhua Han
  - Amir H. Abdi
  - Dongsheng Li
  - Chin-Yew Lin
  - Yuqing Yang
  - Lili Qiu
tags:
  - 2024
  - paper
status: complete
importance: high
---

# MInference 1.0: Accelerating Pre-filling for Long-Context LLMs via Dynamic Sparse Attention

**Huiqiang Jiang, Yucheng Li, Chengruidong Zhang, Qianhui Wu, Xufang Luo, Surin Ahn, Zhenhua Han, Amir H. Abdi, Dongsheng Li, Chin-Yew Lin, Yuqing Yang, Lili Qiu** (Microsoft Corporation; Yucheng Li is at the University of Surrey, work done during a Microsoft internship; first three authors equal contribution) — [arXiv:2407.02490](https://arxiv.org/abs/2407.02490), cs.CL, v1 2 Jul 2024, v2 30 Oct 2024. Accepted at **NeurIPS 2024 (Spotlight)**. Code: `https://aka.ms/MInference`. Same group as LLMLingua / LongLLMLingua ([[prompt-compression]]).

## Summary

The prefill of a long prompt is quadratic in the attention term, and for an 8B model that term becomes the whole bill: LLaMA-3-8B on a single A100 keeps the user waiting **6 minutes for a 300K-token prompt and 30 minutes for 1M tokens**, with self-attention exceeding **90% of total pre-filling latency** at those lengths (Fig. 2a). Attention weights are known to be sparse — in a 128K×128K attention matrix, keeping only the top-4K columns recalls **96.8%** of total attention mass — but the sparsity is *dynamic*: reusing those same top-4K column indices on a different 128K prompt drops recall to **83.7%**. That rules out fixed patterns (Longformer, BigBird) without retraining, and the prior dynamic estimators (low-rank projections, post-hoc statistics) cost too much at long context. MInference's bet is that while the *indices* move, the *shape* of each head's sparsity is stable per head, so it can be classified offline and only the indices need to be estimated online.

The mechanism has three steps. (1) **Offline kernel-aware pattern search** assigns each head one of three patterns — **A-shape** (initial tokens + local window, the [[attention-sink]] / StreamingLLM pattern, static), **Vertical-Slash** (a few full columns plus a few diagonals, whose positions move with the input), or **Block-Sparse** (dispersed but spatially clustered 64×64 blocks) — by comparing candidate settings with FLOPs-matched cost *as measured in the GPU kernel*, choosing the one with the highest recall of the attention *output* (so V is included) on a single 30K-token KV-retrieval sample (~15 min on one A100). (2) **Online index building** per prompt: A-shape needs no index; Vertical-Slash multiplies the *last 64 queries* against all keys, sums the resulting estimated attention along columns and along diagonals, and takes the top-k_v verticals and top-k_s slashes; Block-Sparse mean-pools Q and K in blocks of 64, computes a block-level attention map, and keeps the top-100 blocks. (3) **Sparse attention with custom Triton kernels** built on FlashAttention and the PIT dynamic-sparse compiler: a block-sparse FlashAttention kernel whose latency is linear in the number of retained blocks, and a hybrid Vertical-Slash kernel that treats slashes as 64×64 blocks and verticals as 1×64 columns gathered via PIT. After the search, **more than 90% of heads are Vertical-Slash**; Block-Sparse heads cluster in intermediate-to-later layers and A-shape heads in middle layers.

Results: on LLaMA-3-8B on a single A100 (bf16), end-to-end prefill speedups of **1.8× at 100K, 4.1× at 300K, 6.8× at 500K, and 10× at 1M** — 30 minutes down to 3 minutes per 1M-token prompt, or 22 s on 8×A100 with tensor+context parallelism. Quality holds: InfiniteBench averages are within +0.2 to +0.8 of full attention on LLaMA-3-8B-262K, Yi-9B-200K, GLM-4-9B-1M and LLaMA-3-70B-262K, RULER *improves* for LLaMA-3-8B-262K (84.4 → 87.0, effective context 16K → 32K) and GLM-4-9B-1M (88.0 → 89.6), PG-19 perplexity at 100K is 0.2 above dense, and Needle-in-a-Haystack stays intact to 1M. The method needs no retraining. It breaks down where its assumptions do: below ~10K tokens the index-building overhead swells from 5% to 30% and latency converges to FlashAttention; Yi-9B-200K loses 3.4 points on RULER; pushing sparsity higher degrades accuracy; and static (non-dynamic) indices collapse KV-retrieval to near zero.

## Key points

**Where this sits: attention sparsification, not token pruning.** MInference **keeps every token in the sequence and every entry of the KV cache**; it skips most *entries of the attention matrix* (QKᵀ and PV) during prefill. It is therefore orthogonal to *token pruning* (dropping tokens from the sequence before or during prefill, e.g. [[prompt-compression]] / LLMLingua from the same authors) and to *speculative prefill* (a draft model choosing which tokens to drop), and it composes with decode-side KV compression (the paper stacks it with SnapKV). It only touches the **O(N²) attention term** — the Q/K/V/O projections and the FFN are computed densely for all N tokens — so its end-to-end gain is bounded by attention's share of prefill time. Decoding is left fully dense.

**The three patterns** (Table 1; block sizes from Fig. 3c/Fig. 7):

| Pattern | Spatial distribution | Index build | Kernel latency | Online estimate | Search space (Table 7) |
|---|---|---|---|---|---|
| A-shape | static structured: initial tokens + local window | zero | low | none | {(1024 global, 4096 local)} |
| Vertical-Slash | dynamic structured: columns + diagonals | small | medium | last_q = 64 queries × all keys → column sums, diagonal sums → top-k_v, top-k_s | {(30, 2048), (100, 1800), (500, 1500), (3000, 200)} as (k_v, k_s) |
| Block-Sparse | dynamic structured, clustered 64×64 blocks | small | low | mean-pool Q, K in 64-blocks → block attention → top-k_b | {100} blocks |
| Top-K (prior work) | dynamic fine-grained | high | high | — | — |

All candidates are FLOPs-matched to the A-shape budget (1K global + 4K local); "kernel-aware" means the FLOPs are those the kernel actually executes after block coverage, not a conceptual count. Nearest-non-zero distances in a 128K attention map concentrate around 5 across layers and heads, which is the evidence for the block-clustering assumption. The same head assignment found on LLaMA-3-8B-262K is reused unchanged for LLaMA-3-8B-1M.

**Kernels.** Block-sparse FlashAttention (Triton port of FlashAttention-2): each thread block loops over its row's selected blocks; speedup over dense FlashAttention is approximately `S / (2·B·k_b)` (Eq. 3). Vertical-Slash uses two kernels: an index kernel that merges vertical *points* and slash *ranges* per block-row in O(k_v + k_s), then a hybrid FlashAttention kernel that first loops the merged blocks (block-sparse part) and then loops the leftover columns grouped by 64 (PIT gather part); latency is linear in the total area of blocks plus columns. Since everything is Triton, the authors expect similar speedups on H100 / MI300X but only report A100.

**End-to-end prefill latency — LLaMA-3-8B, single A100, bf16** (Fig. 1b):

| Context | 100K | 300K | 500K | 1M |
|---|---|---|---|---|
| Speedup vs. dense (FlashAttention) prefill | 1.8× | 4.1× | 6.8× | **10×** |

1M tokens: 30 min → 3 min on one A100; 22 s on 8×A100 with tensor + context parallelism. Dynamic index building is **5–20%** of the remaining time; the rest is the sparse attention itself.

**Attention-kernel speedup vs. end-to-end speedup** (Appendix D.2, Fig. 10 and 12; single A100, 1M context):

| Kernel | Speedup vs. FlashAttention at 1M | Index-build share of kernel time |
|---|---|---|
| Vertical-Slash (slowest of the three) | **13×** | 5–15% |
| A-shape | 50% slower than Vertical-Slash at 1M (164 ms) | 0% |
| Block-Sparse (fastest) | **30×** | ~25% (MeanPooling + block matmul) |

Kernel-side sparsity (fraction of the causal FlashAttention work actually skipped) exceeds **90% beyond 200K** tokens (≥8× kernel speedup even with 20% index overhead) and **95% beyond 500K** (theoretical >15×). Index memory stays under 160 MB for LLaMA-3-8B at 1M. At 10K all four kernels, including dense FlashAttention, take under 1 ms and are nearly indistinguishable.

Reading the two tables together: the paper's own figure is that attention is ">90%" of dense prefill at 300K–1M. With 13–30× kernels, a 10× end-to-end number is consistent with attention being ~97% of the 1M prefill (derived, not stated — but it matches the FLOP arithmetic: 4·N²·d·L for attention vs. 2·N·params for projections+FFN puts attention at ~97% for an 8B model at N = 1M and ~75% at N = 100K). At 100K the linear-term share is larger *and* the kernel sparsity is below the 90% it reaches past 200K, which is why the end-to-end gain is only 1.8× there. The projection/FFN work — the compute-bound ~1,024–1,215 OPs/byte part of prefill in the [[llm-inference-roofline-yuan-2024]] analysis — is untouched, so this family's payoff is strictly a long-context payoff.

**Quality retention — InfiniteBench** (Table 2, 6; avg over 10 tasks, ~214K avg context; Retr.KV shown as the most sparsity-sensitive task):

| Model | Full attention | StreamingLLM (A-shape only) | InfLLM | Ours w/ static indices | **MInference** | Retr.KV full → ours |
|---|---|---|---|---|---|---|
| LLaMA-3-8B-Instruct-262K | 38.2 | 23.8 | 34.8 | 31.9 | **38.8** | 14.4 → 12.8 |
| Yi-9B-200K | 37.5 | 16.8 | 31.9 | 22.9 | **37.7** | 28.8 → 27.6 |
| GLM-4-9B-1M | 46.7 | 27.0 | 37.3 | — | **47.0** | 41.0 → 43.0 |
| LLaMA-3-70B-Instruct-262K | 46.5 | 21.6 | 39.2 | — | **47.3** | 34.0 → 39.0 |

StreamingLLM with dilated or strided windows is worse still (24.2 / 13.3 on LLaMA-3-8B; 4.2 / 4.6 on Yi-9B). Static indices drop Retr.KV to 0.2 (LLaMA) and 1.0 (Yi).

**Quality retention — RULER** (Table 3; 4K–128K, 2,600 examples per length; "effective" = length at which score ≥ 85%):

| Model | Method | Effective ctx | 4K | 8K | 16K | 32K | 64K | 128K | Avg |
|---|---|---|---|---|---|---|---|---|---|
| LLaMA-3-8B-262K | Full | 16K | 97.2 | 91.8 | 87.3 | 80.8 | 77.4 | 72.2 | 84.4 |
| | MInference | **32K** | 97.7 | 91.2 | 88.5 | 85.0 | 82.3 | 77.6 | **87.0** |
| | InfLLM | 4K | 89.4 | 79.8 | 70.1 | 55.6 | 43.0 | 39.5 | 62.9 |
| Yi-9B-200K | Full | 8K | 91.9 | 90.2 | 78.8 | 76.3 | 68.1 | 62.9 | 78.1 |
| | MInference | 8K | 92.3 | 89.7 | 79.0 | 73.8 | 64.7 | 56.9 | **74.7** |
| GLM-4-9B-1M | Full | 64K | 93.8 | 91.6 | 89.3 | 87.4 | 85.2 | 80.8 | 88.0 |
| | MInference | 64K | 94.6 | 93.1 | 91.0 | 89.6 | 85.5 | 84.0 | **89.6** |

The gain on LLaMA-3-8B-262K beyond 32K is real but the Yi-9B loss (−3.4 avg, −6.0 at 128K) shows retention is model-dependent. PG-19 language modeling with 100K prompts: perplexity **0.2 above** full attention, and 0.25 (Yi-9B) / 0.75 (LLaMA-3-262K) *below* StreamingLLM. Needle-in-a-Haystack on LLaMA-3-8B-1M is retained from 1K to 1M; StreamingLLM and InfLLM fail once the needle leaves the global-token/local-window range. NIAH also holds on GLM-4-9B-1M, Yi-9B-200K, Phi-3-Mini-128K and Qwen2-7B-128K (Appendix D.1), with slight *gains* near 100K on Yi and Phi-3.

**Ablations** (Tables 4 and 8; LLaMA-3-8B-262K, InfiniteBench avg):

| Variant | Avg | Retr.KV |
|---|---|---|
| MInference (all three patterns, dynamic) | **38.8** | 12.8 |
| only Vertical-Slash | 37.1 | 5.0 |
| only slash lines (+ top-1 vertical) | 35.9 | 4.2 |
| static Vertical-Slash / Block-Sparse indices | 31.9 | 0.2 |
| only A-shape (= StreamingLLM) | 23.8 | 0.8 |
| only Block-Sparse | 18.7 | 0.0 |
| only vertical lines (+ top-1 slash) | 18.6 | 0.0 |

Dynamic index estimation is the load-bearing piece; slash lines carry more than verticals; Block-Sparse alone is nearly as bad as verticals alone even though >90% of heads *prefer* Vertical-Slash in the search.

**Composition with decode-side KV compression** (Table 5): LLaMA-3-8B-262K with SnapKV at decode scores 36.0 on InfiniteBench; MInference prefill + SnapKV decode scores 37.3. The combination is the paper's argument that attention sparsification (prefill) and KV eviction (decode) are separate levers.

**Where it breaks**

- *Short prompts.* At 10K context, index building rises from 5% to 30% of time and end-to-end latency "approaches that of FlashAttention" (Appendix A). The speedup curve only clears 2× somewhere past 100K on this hardware.
- *Higher sparsity.* The authors note accuracy "may noticeably decline" if the sparsity rate is raised beyond the searched budget; the reported numbers are at the 1K + 4K A-shape-equivalent FLOPs budget.
- *Model dependence.* Yi-9B-200K loses RULER points at every length ≥ 32K; En.Dia on LLaMA-3-8B moves 6.0 → 7.5 while En.MC dips 67.3 → 65.9 — the "matches or surpasses" phrasing is an average, not a per-task guarantee.
- *Kernel constraints.* The Vertical-Slash kernel must retain at least one vertical and one slash line; slash lines are computed at 64×64 block granularity so a single diagonal costs a full block-diagonal band.
- *Calibration.* Head patterns come from one 30K KV-retrieval sample; the paper reports it generalizes across lengths and domains, but there is no sensitivity study of the search sample.
- *Memory.* Nothing shrinks: the full KV cache is still produced, so this does not help the decode-side memory wall.
- *Scope.* Causal decoder-only LLMs with RoPE; the appendix shows vertical/slash structure also in T5-style encoders (Flan-UL2), BERT and multimodal LLMs but does not evaluate them.

## Related

- [[sparse-attention]] — the family this paper anchors (no concept page yet; needed as the umbrella for A-shape / Vertical-Slash / Block-Sparse and for the trained variants below).
- [[attention-mechanism]] / [[scaled-dot-product-attention]] / [[multi-head-attention]] — what is being sparsified: per-head QKᵀ softmax and PV, with the mask applied inside the kernel (Eq. 1).
- [[flash-attention]] — the dense baseline and the kernel MInference is built on; [[flash-attention-dao-2022]], [[flash-attention-2-dao-2023]] for the block-sparse FlashAttention lineage the Eq. 3 speedup model comes from.
- [[attention-sink]] — the A-shape pattern is exactly StreamingLLM's sink + local window; used as one of three patterns and as the FLOPs reference budget.
- [[kv-cache]] — untouched by MInference; the paper composes with SnapKV eviction at decode time.
- [[long-context-llm]] — the regime (≥100K) where attention dominates prefill and this method pays off.
- [[llm-inference-roofline-yuan-2024]] — why the linear projection/FFN term bounds the end-to-end gain at shorter contexts.
- [[methods-to-speed-up-prompt-processing]] — the wiki's synthesis; this is the training-free, attention-only entry in the "cut the quadratic term" tier.
- [[minimax-sparse-attention-lai-2026]] and [[flashmemory-lookahead-sparse-attention-wang-2026]] — trained block-sparse / lookahead-index successors; MInference is the retrofit-without-training point on the same axis.
- [[token-sparse-attention-jo-2026]] and [[claa-cross-layer-attention-aggregation-mcdanel-2026]] — later sparse-attention prefill methods in this ingest group; see those pages for how they differ on what is skipped.
- [[prompt-compression]] — the *token pruning* alternative (LLMLingua family, same authors), which does reduce the FFN term but changes the input.
- [[chunked-prefill]] / [[prefix-caching]] — orthogonal systems-level prefill levers.
- [[linear-attention]] / [[state-space-model]] — the architectural alternatives the related-work section contrasts, which require training from scratch.
- [[microsoft]] / [[microsoft-research]] — authors' affiliation; [[llama]] and [[qwen]] — model families evaluated (Gradient AI's LLaMA-3-8B/70B long-context variants; Qwen2-7B-128K on NIAH).

## References

- [arXiv:2407.02490](https://arxiv.org/abs/2407.02490) — v2, 30 Oct 2024; NeurIPS 2024 Spotlight.
- Code: https://aka.ms/MInference

## Related 2026 sources

- [[trianglemix-decoding-time-contribution-sparsity-he-2025]] — the same group's follow-up: a static pattern that stacks on MInference for a further 19–24% TTFT at 128K; also measures MInference at **0.5× at 32K**
- [[uniprefill-block-wise-dynamic-sparsification-fan-2026]] — beats MInference on speed at 128K (2.26× vs 1.34×) but MInference stays *more accurate* on RULER (90.68 vs 90.45)
- [[trained-sparse-attention]] — the trained alternative to retrofits like this one
