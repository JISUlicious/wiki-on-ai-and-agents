---
title: "UniPrefill: Universal Long-Context Prefill Acceleration via Block-wise Dynamic Sparsification"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2605.06221"
authors:
  - Qihang Fan
  - Huaibo Huang
  - Zhiying Wu
  - Bingning Wang
  - Ran He
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# UniPrefill: Universal Long-Context Prefill Acceleration via Block-wise Dynamic Sparsification

**Qihang Fan, Huaibo Huang, Zhiying Wu, Bingning Wang, Ran He** (MAIS & NLPR, CASIA; UCAS; WeChat, Tencent — first author's work "done during internship at WeChat"; the HTML marks a corresponding author and a project leader but the marks do not survive rendering) — [arXiv:2605.06221](https://arxiv.org/abs/2605.06221), cs.CL, submitted 7 May 2026, v1 only, 8,458 KB. **No venue stated.** **Code released:** [github.com/qhfan/UniPrefill](https://github.com/qhfan/UniPrefill) — a fork of vLLM v0.16.0 plus a `native_imp.py`; the README lists LLaMA-3.1, Qwen3-Next and Gemma3 as supported and names no GPU.

> [!important] What this is, in one paragraph
> **Token pruning, not attention sparsification, engineered for [[vllm|vLLM]] and for hybrid architectures.** At every full-attention layer the last *n* = 128 queries are scored against all keys, the attention mass is pooled into 64-token blocks, and the smallest set of blocks holding **top-*p* = 0.99** of the mass is kept; the dropped blocks then leave the token stream, so *every* downstream sublayer — full attention, linear attention, sliding-window attention, FFN/MoE — runs on fewer rows. Training-free, no weight changes. The paper's pitch is that sparse attention ([[minference-dynamic-sparse-attention-jiang-2024|MInference]], FlexPrefill, XAttention, ProxyAttn) can only touch the full-attention layers, which are ≤ 25 % of a 3:1 linear/full hybrid, whereas token dropping cuts GEMM FLOPs in all of them. That argument is correct — and yet the dense model still gains the most (**2.26× TTFT at 128K on LLaMA-3.1-8B vs 1.68× on Qwen3-Next-80B-A3B and 1.49× on Gemma-3-12B**), for a reason the paper never states (see § Why the hybrids gain less). Read the abstract's "up to 2.1×" as a **throughput** number (+109 % at batch 16), not the TTFT table's 2.26×.

## Problem

Two complaints about the sparse-attention line of prefill work ([[sparse-attention]]):

1. **Architectural mismatch.** Production models are increasingly hybrids — linear/full (Qwen3-Next, [[kimi-linear-kimi-team-2025|Kimi Linear]], [[qwen3.8-flash-next]]) or sliding-window/full (Gemma 3). Sparse attention accelerates only the full-attention operation, so "in a linear/full attention hybrid with a 3:1 ratio, at most one out of every four layers can be accelerated … leaving the dominant computational budget entirely untouched." Its measured consequence is Table 1: on the two hybrids, MInference / FlexPrefill / XAttention / ProxyAttn deliver **1.02–1.11× at 128K and 0.96–0.99× at 4K–8K** — essentially nothing.
2. **Continuous-batching incompatibility.** FlexPrefill-style methods "operate on individual requests in isolation and assume static batch composition", so they have stayed research prototypes rather than being embedded in [[vllm|vLLM]].

The observation the fix rests on is the same one every [[token-pruning]] paper uses: next-token prediction depends only on the final hidden state of the last position, so a prompt token whose attention weight from the last queries is ≈ 0 at a given layer can be dropped from the layers after it. The paper's own framing of what is new is that the drop decision is (a) made at full-attention layers and *propagated* to non-attention sublayers, and (b) implemented as a continuous-batching operator with tensor-parallel support.

## Mechanism

**Scoring (§ 3.2).** For block *b* (a full-attention layer plus its M_b following sublayers), compute the partial GEMM **S = Q[N−n:N] · Kᵀ ∈ ℝ^{n×N}** for the last **n = 128** queries, cost O(nNd_k). Apply an **online softmax over the full key sequence** (so the weights are properly normalised before any pooling), average over the *n* query rows and over heads, then reduce within non-overlapping token blocks of **G = 64** to get one score s̄_g per block. The paper contrasts this with SnapKV, which uses the same observation window but only *after* a full N×N prefill to compress the KV cache for decode.

**Selection (§ 3.3).** Sort blocks by score and keep the minimal prefix whose cumulative share of total mass reaches **p = 0.99** (0.98 for Gemma-3-12B). The **first A = 128 tokens** ([[attention-sink|attention sinks]]) and the **last n = 128 tokens** (the query window) are always kept. Error bound (Eq. 6): the perturbation to any retained position is ≤ (1−p)·max‖v_i‖ — at most 1 % of attention mass is discarded per layer. The authors argue top-*p* over top-*k* because it adapts to how concentrated attention is: a fixed *k* over-retains on peaky heads and under-retains on diffuse ones.

**Propagation (§ 3.4).** Dropped tokens are excluded from every subsequent sublayer — "every full attention, linear attention, sliding window attention, and FFN layer processes only the retained set." FLOPs saved by a drop at layer ℓ₁ with retention ρ scale as **(1−ρ)·(L−ℓ₁)·O(Nd²)** (Eq. 10), versus at most (1−ρ)·O(N²d_k) for sparse attention at that single layer; the ratio (L−ℓ₁)·Nd²/(N²d_k) is claimed to diverge as N → ∞ (Eq. 11 — which is backwards for a quadratic attention term, see § Claims). Error accumulation is bounded by the product of sublayer Lipschitz constants (Eq. 12), which LayerNorm and residuals keep in check "in practice".

> [!warning] Does a drop persist across full-attention layers? The paper says both yes and no
> The cascade is what the FLOPs analysis (Eq. 9–11), the § 3.4 prose ("reduces the token count for all layers ℓ′ > ℓ, including subsequent full attention layers"), the contributions list ("propagates sparsity across all subsequent layers") and the vLLM bookkeeping (metadata patched "to all downstream layers ℓ′ > ℓ"; a per-request drop *history*; Eq. 17 gives each layer the retained length after the most recent drop at or before it) all describe. But **Eq. 8 says the opposite**: "At block b+1, the full sequence is *reconstituted* by carrying dropped token states forward without update" — dropped tokens re-enter with stale hidden states and "importance scores are recomputed fresh at each block's full attention layer" — and the introduction says a dropped token is excluded "in the remaining layers *of the block*", with Fig. 2's caption saying sparsity propagates "within each repeating layer pattern". Under the Eq. 8 reading a token can be revived at the next full-attention layer (like [[probe-and-skip-self-predictive-token-skipping-wu-2026|SPTS]]'s residual pass-through) and the compounding of 32 successive top-0.99 cuts on LLaMA is avoided; under the cascade reading the K/V of a dropped token is never written at any later layer, which is the only way Eq. 17's decode rule makes sense. The public code is a vLLM fork whose README does not settle it. **This page treats the systems description (cascade) as authoritative for the speed numbers and flags the method section as inconsistent.** Either way it is token dropping — every dropped token skips the FFN — which is the property that matters against [[sparse-attention]].

**Kernels and vLLM (§ 3.5).** Four fused Triton kernels over the packed variable-length token stream (indexed by `cu_seqlens`, no padding): partial GEMM with inline causal mask → two-pass online softmax → block reduce over heads and positions → on-GPU top-*p* that packs each (score, index) into one int64 via a monotone IEEE-754 bitcast, sorts, cumsums, thresholds and scatters the keep mask back; a final expansion kernel lifts the mask from block to token granularity and forces sinks on. Under **[[tensor-parallelism|tensor parallelism]] of degree T** each rank sees 1/T of the heads, so partial block scores are all-reduced (Eq. 15) before top-*p* so all ranks make the same drop. Inside vLLM's scheduler a drop event patches `query_start_loc`, `seq_lens` and `num_actual_tokens` for all later layers, recomputes physical KV slots per layer from the retained tokens' logical positions and that layer's block table (Eq. 16 — global and sliding-window layers may have different tables), and injects a per-layer `seqused` correction before each decode step so each attention layer sees exactly the KV entries it wrote (Eq. 17) — "without any modification to model weights or the PagedAttention memory allocator". Dropped tokens therefore have **no KV entries** at the layers after their drop, for the whole generation — the standard [[sparse-prefill]] caveat about multi-turn and later-step retrieval applies and is not discussed.

## Hardware and setup

| | |
|---|---|
| Serving stack | [[vllm\|vLLM]] **v0.16.0** fork, fused Triton kernels, **CUDA 12.8**, **TP = 8** for all throughput runs |
| GPU | **never named** — no A100/H100/H800 anywhere in the paper, the arXiv metadata or the README; only "CUDA 12.8", i.e. [[nvidia]] hardware of unstated generation |
| Accuracy / TTFT (Table 1) | HuggingFace Transformers, **batch size 1** |
| Throughput (Table 2) | vLLM, batch 1 / 4 / 16 / 64, 4K–128K (64K and 128K not run at batch 64) |
| Models | **LLaMA-3.1-8B-Instruct** (32 full-attention layers); **Qwen3-Next-80B-A3B** (48 layers, 3 Gated-DeltaNet : 1 full attention → 12 drop points, MoE); **Gemma-3-12B** (48 layers, 5 sliding-window : 1 global → 8 drop points) |
| Hyperparameters | *p* = 0.99 / 0.99 / **0.98**, G = 64, n = 128, 128 sink tokens |
| Benchmark | [[ruler\|RULER]] 4K–128K (no LongBench, no reasoning benchmark) |

## Results

### Accuracy — RULER (Table 1, HF, batch 1)

Average over 4K–128K, with the per-length deltas for UniPrefill:

| Model | Dense avg | **UniPrefill avg** | Δ | Δ at 4K / 8K / 16K / 32K / 64K / 128K |
|---|---|---|---|---|
| LLaMA-3.1-8B-Instruct | 90.36 | **90.45** | **+0.09** | −0.83 / −0.15 / +0.79 / −1.25 / −1.01 / **+2.98** |
| Qwen3-Next-80B-A3B | 94.76 | **93.94** | **−0.82** | −0.16 / −1.18 / −0.78 / −0.75 / −1.38 / −0.68 |
| Gemma-3-12B | 79.99 | **78.87** | **−1.12** | −0.83 / +0.64 / +0.49 / −1.68 / **−2.57** / **−2.84** |

The dense model is lossless on average (and +3.0 at 128K, the usual distractor-removal effect); the two hybrids lose ~1 point on average, Gemma up to 2.8 at 64K–128K. The headline "no significant accuracy degradation" is fair for LLaMA and mildly generous for Gemma.

### Speed — TTFT speedup over dense (Table 1, HF, batch 1)

| Model | 4K | 8K | 16K | 32K | 64K | 128K |
|---|---|---|---|---|---|---|
| LLaMA-3.1-8B (full attention) | **1.21×** | 1.34× | 1.37× | 1.62× | 2.01× | **2.26×** |
| Qwen3-Next-80B-A3B (linear/full 3:1) | **1.08×** | 1.21× | 1.24× | 1.39× | 1.42× | **1.68×** |
| Gemma-3-12B (SWA/full 5:1) | **1.15×** | 1.21× | 1.22× | 1.26× | 1.31× | **1.49×** |

The ranges **1.21–2.26× / 1.08–1.68× / 1.15–1.49×** check against the table.

### Against the baselines (Table 1)

| Method | Family | LLaMA avg · 128K TTFT | Qwen3-Next avg · 128K TTFT | Gemma avg · 128K TTFT |
|---|---|---|---|---|
| Dense | — | 90.36 · 1.00× | 94.76 · 1.00× | 79.99 · 1.00× |
| [[lazyllm-dynamic-token-pruning-fu-2024\|LazyLLM]] | token pruning | 68.50 · **2.51×** | 69.98 · 1.74× | 67.93 · 1.64× |
| [[sliminfer-dynamic-token-pruning-long-2025\|SlimInfer]] | token pruning | 68.87 · 2.07× | 68.55 · 1.56× | 68.83 · 1.52× |
| [[minference-dynamic-sparse-attention-jiang-2024\|MInference]] | sparse attention | **90.68** · 1.34× | 94.31 · 1.05× | 79.25 · 1.03× |
| FlexPrefill | sparse attention | 89.62 · 1.46× | 93.97 · 1.08× | 78.64 · 1.04× |
| XAttention | sparse attention | 89.34 · 1.38× | 93.53 · 1.05× | 78.26 · 1.02× |
| ProxyAttn | sparse attention | 90.14 · 1.79× | 93.88 · 1.11× | 78.79 · 1.06× |
| **UniPrefill** | token pruning | 90.45 · 2.26× | 93.94 · 1.68× | 78.87 · 1.49× |

Read carefully: UniPrefill is **not** the most accurate method on LLaMA (MInference's 90.68 average beats it, as does MInference's per-length score at 64K), and **not** the fastest at 128K (LazyLLM's 2.51×). It is the best *tradeoff* — the only method in the table that is both ≥ 2× and within a point of dense on LLaMA, and the only one that gets any meaningful speedup on the hybrids. The sparse-attention rows are the wiki's [[sparse-prefill|attention-share]] rule tabulated: on LLaMA they are **below 1.0× at 4K–8K** (MInference 0.82× / 0.86×, ProxyAttn 0.82× / 0.89×), cross 1.0× at 16K–32K, and reach 1.34–1.79× at 128K; on the hybrids they never exceed 1.11×.

> [!warning] The LazyLLM and SlimInfer rows do not look like those methods' own papers
> Here LazyLLM and SlimInfer lose **7–8 RULER points at 4K** (89.16 / 90.23 vs 97.36) and 20–30 points at 128K on LLaMA, and similar on the hybrids. In their own papers, [[sliminfer-dynamic-token-pruning-long-2025|SlimInfer]] is 48.08 → 47.65 on LongBench and [[lazyllm-dynamic-token-pruning-fu-2024|LazyLLM]] is within 0.1 on multi-doc QA at 2.34×. RULER is harder than LongBench and both methods are known to be weak on aggregation tasks, but a 7-point loss at 4K — where SlimInfer's first Slim layer retains 8K tokens and therefore prunes *nothing* — is not credible without the settings, which the paper does not give. Treat the token-pruning baselines as run at unfavourable configurations, not as a fair like-for-like.

### Throughput inside vLLM (Table 2, TP = 8, tokens/s, % over standard prefill)

| Model | batch | 4K | 8K | 16K | 32K | 64K | 128K |
|---|---|---|---|---|---|---|---|
| LLaMA-3.1-8B | 1 | +4 % | +12 % | +20 % | +54 % | +81 % | +107 % |
| | 4 | +6 % | +22 % | +35 % | +59 % | +82 % | +108 % |
| | 16 | +19 % | +33 % | +38 % | +64 % | +87 % | **+109 %** |
| | 64 | +24 % | +35 % | +39 % | **+65 %** | — | — |
| Qwen3-Next-80B-A3B | 1 | **−5 %** | **−4 %** | +3 % | +21 % | +26 % | +48 % |
| | 4 | **−3 %** | +1 % | +16 % | +31 % | +37 % | +57 % |
| | 16 | +2 % | +20 % | +22 % | +38 % | +39 % | **+68 %** |
| | 64 | +13 % | +26 % | +26 % | +43 % | — | — |
| Gemma-3-12B | 1 | **−2 %** | +9 % | +19 % | +25 % | +22 % | +42 % |
| | 4 | +14 % | +29 % | +22 % | +26 % | +30 % | +41 % |
| | 16 | +17 % | +24 % | +23 % | +27 % | +31 % | **+42 %** |
| | 64 | +19 % | +23 % | +22 % | +27 % | — | — |

Absolute dense numbers worth keeping: LLaMA-8B **21,013 tok/s at 128K** (≈ 6.2 s per 128K prompt on 8 GPUs); Qwen3-Next-80B-A3B **33,512 tok/s at 128K but only 15,314 at 4K**; Gemma-3-12B 18,103 at 128K. The **+109 % maximum is at batch 16, not batch 64** — batch 64 was only run to 32K, where its best is +65 %. Gains rise with batch because the fused scoring kernels amortise and because dropping tokens frees the scheduler to pack more prefill into a step; the paper says "scales favorably with request concurrency" without decomposing why.

### Ablations (Tables 3–5, LLaMA and Qwen3-Next)

- **Block size G** (Table 3; the "TTFT speedup" column is byte-identical to Table 2's batch-16 throughput row, so it is a throughput number): G = 128 is fastest at short context (+26 % at 4K on LLaMA vs +19 %) but costs ~2 RULER points; G = 32 is fastest at long context (**+121 %** LLaMA, **+78 %** Qwen3-Next at 128K) but loses 3–4 points at 4K (93.42 / 92.17). G = 64 is the compromise.
- **Query window n** (Table 4, LLaMA): n = 32 → 87.77 avg (−2.7, "too few query tokens introduce high variance"); n = 128 → 90.45; n = 512 → 90.49 at more scoring cost.
- **Seeds** (Table 5): three seeds give *identical* numbers to two decimals at every length — as they must, since nothing in the method is stochastic. The "robust to random seed initialization" appendix is vacuous.

## Why the hybrids gain less than the dense model (derived — the paper does not explain it)

The hybrid argument is true and the ranking is still dense > linear/full > SWA/full. Three reasons, in decreasing order of weight:

1. **Token dropping is quadratic on attention and only linear on everything else.** Keeping a fraction ρ of tokens cuts a GEMM to ρ of its cost but cuts an N² attention layer to **ρ²**. LLaMA-3.1-8B has 32 quadratic layers and, at 128K, attention is ~70 % of an 8B prefill (see [[sparse-prefill]] and the derivation in [[trianglemix-decoding-time-contribution-sparsity-he-2025]]); so most of its bill enjoys the ρ² rate. Qwen3-Next has 12 quadratic layers of 48 and Gemma-3 8 of 48 (its sliding-window layers are O(N·1024)); their bills are mostly linear in N, so the ceiling on their speedup is ≈ 1/ρ̄ rather than ≈ 1/ρ̄². The paper's Eq. 11 sends the *ratio* of token-dropping to sparse-attention savings to infinity as N → ∞ by treating GEMMs as the growing term; for a dense model the opposite is true — the N² term grows fastest — and that is exactly why the dense model benefits most.
2. **Fewer, later drop points.** Drops happen only at full-attention layers: 32 chances on LLaMA (the first at layer 0), **12 on Qwen3-Next with the first at layer 3** (three Gated-DeltaNet layers and their MoE FFNs always run on all N), **8 on Gemma-3 with the first at layer 5**. The retained set shrinks in fewer steps, and a fixed prefix of the network is never accelerated.
3. **The hybrids' baselines are already cheap — and less efficient at small N.** Qwen3-Next dense prefills 128K at 33.5K tok/s, 1.6× faster than LLaMA-8B, so there is less to remove. At 4K it manages only 15.3K tok/s, which says the MoE grouped GEMMs and chunked DeltaNet scans on 8 GPUs are far from saturated at small token counts; shrinking the token count there makes each expert's GEMM smaller and less efficient, which is why UniPrefill is a **net loss on Qwen3-Next at 4K–8K, batch 1** (−5 % / −4 %). Gemma-3 shows the same at 4K (−2 %).

## Short context — the paper's own numbers versus the wiki's rule

[[sparse-prefill]] states that below ~8K token pruning can be a net loss because selection overhead is not amortised. UniPrefill reports both signs. In HF at batch 1 (Table 1) it is positive everywhere — 1.21× / 1.08× / 1.15× at 4K. In vLLM at batch 1 (Table 2) it is **+4 % on LLaMA, −5 % on Qwen3-Next, −2 % on Gemma at 4K, and still −4 % on Qwen3-Next at 8K**; the hybrids only turn positive at batch ≥ 4 (Gemma) or ≥ 16 (Qwen3-Next). So the rule holds for the hybrids in the serving setting, and holds for the *sparse-attention* baselines everywhere (0.82–0.98× at 4K–8K on LLaMA, Table 1). The 8B dense model at 4K is the one case where UniPrefill is a clear small win, presumably because 32 layers of scoring over 4K tokens is cheap relative to 32 layers of N² attention.

## Claims that did not check out, or need qualification

- **"Up to 2.1× speedup in TTFT" (abstract, intro, conclusion)** — the TTFT table's maximum is **2.26×**; 2.1× is the batch-16 **throughput** gain (+109 % ≈ 2.09×) from Table 2, and the intro cites Fig. 1 (a throughput figure) for it. The paper uses "TTFT speedup" and "throughput gain" interchangeably (Table 3's caption vs its contents).
- **"+109 % at batch 64"** — the +109 % cell is **batch 16, 128K**; batch 64 stops at 32K (+65 %).
- **Eq. 8 (reconstitution) vs Eq. 9–11 / § 3.5 (cascade)** — see callout; the method section and the systems section describe different algorithms.
- **Eq. 11's N → ∞ limit** — the ratio (L−ℓ₁)·Nd²/(N²d_k) → 0, not ∞; as written it argues against the paper's own point. The intended claim (absolute GEMM savings are large when N ≫ d) is fine.
- **"Best accuracy-efficiency tradeoff among all methods"** — tradeoff yes; MInference is more accurate on LLaMA (90.68 vs 90.45) and LazyLLM is faster at 128K (2.51× vs 2.26×).
- **"Without observable accuracy degradation" (App. C)** — Gemma-3-12B loses 2.6–2.8 RULER points at 64K–128K and 1.1 on average.
- **LazyLLM / SlimInfer baselines** collapse 7–8 points at 4K with no configuration given — see callout.
- **Hardware** is never named; **Table 1's TTFT** is HuggingFace at batch 1, not the vLLM stack the paper is about.
- Verified: all Table 1 ranges (1.21–2.26×, 1.08–1.68×, 1.15–1.49×), Table 2 maxima (+109 / +68 / +42 %), the negative cells, G / n / p / sink settings, TP = 8, vLLM v0.16.0, CUDA 12.8, 3:1 and 5:1 ratios, Table 3 +121 % / +78 %, Table 4 values.

## Limitations

- **Only RULER.** No LongBench, no reasoning benchmark, no multi-turn — and RULER is the benchmark on which token pruning is known to look best (retrieval-heavy) except for its aggregation tasks, which are not broken out.
- **Dropped tokens have no KV at later layers** (Eq. 17), so any later decode step or follow-up turn that needs them cannot recover them; unlike [[lazyllm-dynamic-token-pruning-fu-2024|LazyLLM]] / [[sliminfer-dynamic-token-pruning-long-2025|SlimInfer]] there is no revival mechanism, and the paper does not discuss decode quality beyond RULER's short answers.
- **Compounding of 32 top-0.99 cuts** on a dense model is never quantified — no retention-ratio curve, no per-layer kept-token counts, so the reader cannot see how much of the sequence survives to the last layer.
- **Method ambiguity** (cascade vs reconstitution) plus **unnamed GPUs** make the speed numbers hard to reproduce from the paper alone.
- Net loss at 4K–8K, batch 1, on both hybrids inside vLLM.
- One p per model, tuned (0.98 for Gemma); the sensitivity of accuracy to p is not shown.
- No comparison against [[discovering-the-gems-in-early-layers-shi-2024|GemFilter]], [[speculative-prefill-liu-2025|SpecPrefill]] or [[probe-and-skip-self-predictive-token-skipping-wu-2026|SPTS]], the other training-free token-droppers, nor against [[chunked-prefill]] scheduling as an alternative route to prefill/decode co-processing.

## Related

- [[sparse-prefill]] — the umbrella; this is the first token-pruning source in the wiki that (a) targets hybrids and (b) runs inside a continuous-batching engine. Its Table 1 is the attention-share rule with eight methods and three architectures in one place.
- [[token-pruning]] — the family; UniPrefill is *static-per-layer, no revival, cascading (per § 3.5)*, selector = last-128-query attention pooled into 64-token blocks with top-*p*. Compare the axes table there.
- [[sparse-attention]] · [[minference-dynamic-sparse-attention-jiang-2024]] — the line it argues against; the hybrid rows of Table 1 are the strongest published evidence that sparse attention does nothing for linear/full and SWA/full models.
- [[trianglemix-decoding-time-contribution-sparsity-he-2025]] — the same authors as MInference, ingested alongside this one; it explains *why* last-query scoring is a valid importance signal (locality of influence) and, from the sparse-attention side, quantifies the same ceiling UniPrefill's hybrid results run into.
- [[lazyllm-dynamic-token-pruning-fu-2024]] · [[sliminfer-dynamic-token-pruning-long-2025]] — the token-pruning baselines; their poor showing here is unexplained (see callout). [[discovering-the-gems-in-early-layers-shi-2024|GemFilter]] and [[claa-cross-layer-attention-aggregation-mcdanel-2026|CLAA]] are the closest single-layer-static relatives that are *not* compared.
- [[speculative-prefill]] · [[speculative-prefill-liu-2025]] · [[draft-based-approximate-inference-galim-2025]] — the draft-model route to the same token dropping.
- [[linear-attention]] · [[state-space-model]] · [[mixture-of-experts]] · [[qwen3.8-flash-next]] · [[kimi-linear-kimi-team-2025]] · [[qwen3.8-flash-next-model-card-2026]] — the hybrid architectures whose prefill this is for; Qwen3-Next-80B-A3B is the 3:1 Gated-DeltaNet/attention MoE from [[qwen]] / [[alibaba]], Gemma-3-12B the 5:1 SWA/global model from [[google]], LLaMA-3.1-8B from [[meta]] ([[llama]]).
- [[vllm]] · [[paged-attention]] · [[chunked-prefill]] · [[prefix-caching]] — the serving stack; the scheduler patches (Eq. 16–17) are the engineering contribution and the reason the KV cache of dropped tokens is simply never allocated.
- [[attention-sink]] · [[massive-activations-attention-sinks-sun-2026]] — the 128 always-kept prefix tokens.
- [[kv-cache]] — smaller by the drop ratio at every layer after a drop; the paper does not report the memory saving.
- [[arithmetic-intensity]] · [[llm-inference-roofline-yuan-2024]] · [[flash-attention]] — why dropping rows from a compute-bound prefill is a FLOP saving rather than a bandwidth one.
- [[methods-to-speed-up-prompt-processing]] — the "shrink it" tier this belongs to.
- [[pop-prefill-only-pruning-he-2026]] — the other way to cut FFN work during prefill: skip whole deep *layers* for the prompt tokens rather than dropping *tokens* from all layers; the two are orthogonal and both training-free. [[sale-low-bit-estimation-sparse-attention-prefill-ji-2025|SALE]] is a further sparse-attention peer whose estimator overhead (24 % of attention time at 8K) is the same short-context tax Table 1's sparse-attention rows show.
- [[ruler]] — **no page**; the benchmark deserves one (RULER is now the standard long-context suite and appears in SlimInfer, TriangleMix, MInference, Token Sparse Attention and here).
- FlexPrefill, XAttention, ProxyAttn, VSPrefill — **no source pages**; cited in References below.

## References

- [arXiv:2605.06221](https://arxiv.org/abs/2605.06221) · code [github.com/qhfan/UniPrefill](https://github.com/qhfan/UniPrefill)
- MInference: Jiang et al., [arXiv:2407.02490](https://arxiv.org/abs/2407.02490) — [[minference-dynamic-sparse-attention-jiang-2024]].
- FlexPrefill: Lai et al., [arXiv:2502.20766](https://arxiv.org/abs/2502.20766), ICLR 2025 — context-aware sparse attention; the "incompatible with continuous batching" example.
- XAttention: Xu et al., [arXiv:2503.16428](https://arxiv.org/abs/2503.16428) — antidiagonal block scoring.
- ProxyAttn: Chen et al., 2025 — proxy-head block attention; the strongest sparse-attention baseline here (1.79× at 128K on LLaMA).
- VSPrefill: Chen, [arXiv:2603.04460](https://arxiv.org/abs/2603.04460) — vertical-slash with lightweight indexing (cited, not compared).
- LazyLLM: Fu et al., [arXiv:2407.14057](https://arxiv.org/abs/2407.14057) — [[lazyllm-dynamic-token-pruning-fu-2024]]. SlimInfer: Long et al., [arXiv:2508.06447](https://arxiv.org/abs/2508.06447) — [[sliminfer-dynamic-token-pruning-long-2025]].
- SnapKV: Li et al., [arXiv:2404.14469](https://arxiv.org/abs/2404.14469) — the observation-window KV compressor the scoring is contrasted with.
- StreamingLLM: Xiao et al., [arXiv:2309.17453](https://arxiv.org/abs/2309.17453) — attention sinks.
- RULER: Hsieh et al., [arXiv:2404.06654](https://arxiv.org/abs/2404.06654).
- Qwen3-Next: Qwen team 2025; Gemma 3: Gemma team, [arXiv:2503.19786](https://arxiv.org/abs/2503.19786); Llama 3: Grattafiori et al., [arXiv:2407.21783](https://arxiv.org/abs/2407.21783).
- vLLM / PagedAttention: Kwon et al., [arXiv:2309.06180](https://arxiv.org/abs/2309.06180) — [[paged-attention-kwon-2023]]; Orca (continuous batching): Yu et al., OSDI 2022.
