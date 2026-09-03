---
title: "Probe and Skip: Self-Predictive Token Skipping for Efficient Long-Context LLM Inference (SPTS)"
type: source
created: 2026-09-03
updated: 2026-09-03
sources: []
arxiv_id: "2601.13155"
year: 2026
authors:
  - Zimeng Wu
  - Donghao Wang
  - Chaozhe Jin
  - Jiaxin Chen
  - Yunhong Wang
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Probe and Skip: Self-Predictive Token Skipping for Efficient Long-Context LLM Inference (SPTS)

**Zimeng Wu\*, Donghao Wang\*, Chaozhe Jin, Jiaxin Chen, Yunhong Wang** (Beihang University, State Key Laboratory of Virtual Reality Technology and Systems; \*equal contribution) — [arXiv:2601.13155](https://arxiv.org/abs/2601.13155), cs.CL, v1 19 Jan 2026, v2 2 Feb 2026. ICML-format preprint with an Impact Statement; "source code upon acceptance" — no venue, no code yet.

## Summary

SPTS reframes the prefill token-reduction problem as **token skipping** rather than token pruning. Skipping exploits the residual structure Y = X + F(X): a token that is not selected for a block simply passes through the shortcut unchanged (Eq. 2), so it stays in the sequence and can be selected again by a later block. Pruning, by contrast, deletes the token. The paper's critique of prior work is three-fold: (1) methods target either attention (MInference, FlexPrefill) or the FFN (FTP) but not both; (2) selection uses **"outdated" criteria** — attention scores from *preceding* layers ([[lazyllm-dynamic-token-pruning-fu-2024]], [[sliminfer-dynamic-token-pruning-long-2025]]) that do not reflect importance in the layer about to be computed; (3) keeping every token as a candidate through deep layers dilutes softmax scores (Fig. 3c: 95% attention coverage needs ever more tokens as length grows) and destabilises top-k selection (Fig. 3d: Jaccard overlap of consecutive-layer selections falls with length). Its empirical anchor is Fig. 3b: on LLaMA 3.1 8B, disabling deep-layer MHA updates for *every token except the last* barely moves accuracy, while disabling the last token's update is catastrophic — deep attention mainly aggregates into the last position.

The principle — "a token's eligibility for skipping is determined by its potential impact within the *target* layer" — is instantiated per block. **Partial Attention Probing (PAP)** for MHA: at a skip layer compute the key projection for *all* candidate tokens but the query for the *last token only*; the head-averaged softmax attention of that one query (Eq. 4) is the contribution score; the top-M tokens get Q and V projections, a reduced causal attention, and the output projection, and **only their K/V are written to the cache** at that layer. **Low-rank Transformation Probing (LTP)** for the FFN: an offline-built proxy FFN — keep the top-D_low intermediate channels by a saliency score (mean of the top-ρ=0.2 fraction of |σ(xW_gate)⊙xW_up| over 200 calibration sequences from LongBench-Qasper), then SVD each reduced matrix to rank r — predicts each token's transformation magnitude ||f(X_n)||₂; that is multiplied by the PAP score (Eq. 14) so a token is skipped only if it is both uninformative *and* barely changed. **Multi-Stage Delayed Pruning (MSDP)** splits the skip layers into stages with a fixed *token budget* per stage (not a ratio), and at each stage boundary removes tokens from the candidate set outright using the last PAP scores. Settings for LLaMA 3.1 8B (32 layers): skipping from layer 10, stage ends after layers 13/18/23/28, active budgets (9K, 7K, 4K, 2K), candidate set cut by 1K per stage; Qwen2.5 7B: from layer 9, ends 12/16/20/24, budgets (13K, 10K, 7K, 4K), cut 2K per stage; openPangu-Embedded-1B: from layer 11, ends 13/16/19/22, budgets (13K, 10K, 7K, 4K), cut 2K. Proxy sizes (D_low, r) = (512, 192), (1024, 256), (1024, 128). Crucially, **all of this runs in the prefill phase only** (Algorithm 1, line 4: `skip ← Is_prefill_phase() ∧ In_skip_layers(l)`); decoding is standard attention over the compressed, layer-ragged cache.

On the 17 LongBench datasets, at "comparable average TTFT acceleration ratios," SPTS averages **47.80 on LLaMA 3.1 8B (full model 47.98) at 1.68× TTFT**, versus SlimInfer 47.56 at 1.62×, LazyLLM 47.13 at 1.63×, GemFilter 47.09 at 1.31×; on Qwen2.5 7B 47.50 vs. 47.76 at 1.36×; on openPangu-1B (Ascend 910B2 NPU) 30.62 vs. 31.09 at 1.32×. The margins over the second-best method are **0.24, 0.07 and 1.13 points**. On sampled 32K inputs (single A800) the headline is **2.46× TTFT and 2.29× end-to-end** (16 decoded tokens) — the E2E figure is the paper's real claim, since LazyLLM and SlimInfer must recompute revived tokens during decode whereas SPTS's decode simply attends to a smaller cache. The ablation isolates contributions cleanly: PAP alone 1.44× TTFT at 32K, +LTP 2.15×, +MSDP 2.46×; PAP cuts KV memory 29.7-58.6%; the proxy FFN adds +0.54 avg over attention-only FFN skipping; delayed pruning adds +1.69 avg.

## Key points

**Where this sits among prefill token-pruning methods** — the four distinctions the wiki's synthesis depends on:

| Question | SPTS |
|---|---|
| Who selects tokens? | The **target model itself**, probing the *current* block: for MHA, the layer's own full key projection against the last token's query (softmax, head-averaged); for FFN, a low-rank proxy distilled from that layer's own FFN weights (offline saliency + SVD, no gradient training) times the MHA score. No draft model; selection cost is one K projection over all candidates plus ~855K FLOPs/token/projection for the proxy (LLaMA default). |
| Which layer selects? | **Every skip layer selects for itself** (layers 10-32 on LLaMA), with per-stage token budgets 9K→7K→4K→2K; stage boundaries at layers 13/18/23/28 also prune the candidate set. Early layers (1-9) run dense. |
| Static or dynamic? | **Static with respect to decoding** — skipping and pruning happen only during prefill; the resulting cache is fixed and decode is standard. *Within* prefill selection is layer-wise: a token skipped at layer l can be active at layer l+1 (Appendix B), until a stage-end prune removes it for good. No revival during decode. |
| Dropped tokens' KV? | **Never computed at the layers where a token was skipped, and never recoverable after prefill.** Each layer caches only its active tokens, so the KV cache is ragged across layers (Fig. 5c). Stage-pruned tokens have no KV at any later layer. |
| Prefill FLOPs reduced? | **Yes** — Q/V projections, attention, output projection and the FFN run only on the M active tokens; only the K projection touches all candidates. 2.46× TTFT at 32K, 1.35× at 8K. Decode is also faster (2.29× E2E at 32K) because the cache is smaller and nothing is recomputed. |

**LongBench averages and TTFT ratio (Table 1; TTFT ratio averaged over LongBench prompts, not the 32K setting)**:

| Method | LLaMA 3.1 8B avg / TTFT | Qwen2.5 7B avg / TTFT | openPangu-1B avg / TTFT |
|---|---|---|---|
| Full model | 47.98 / 1.00× | 47.76 / 1.00× | 31.09 / 1.00× |
| LazyLLM | 47.13 / 1.63× | 45.48 / 1.28× | — |
| MInference | 47.55 / **0.47×** | 47.18 / **0.41×** | — |
| FlexPrefill | 46.60 / 0.96× | 44.50 / 0.90× | — |
| SlimInfer | 47.56 / 1.62× | 47.43 / 1.34× | — |
| PyramidInfer | 47.27 / 1.19× | 45.84 / 1.16× | 29.49 / 0.83× |
| FTP (token skipping) | 42.06 / 1.32× | 44.25 / 1.36× | 26.76 / 0.87× |
| GemFilter | 47.09 / 1.31× | 45.06 / 1.30× | 29.25 / 1.24× |
| **SPTS** | **47.80 / 1.68×** | **47.50 / 1.36×** | **30.62 / 1.32×** |

Two things this table establishes independently of SPTS itself: sparse-attention prefill methods (MInference, FlexPrefill) are **slower than dense FlashAttention** on LongBench-length prompts, and pure token skipping without good selection (FTP) costs ~6 points. Note that the Qwen2.5 "Full Model" and "LazyLLM" rows are digit-for-digit identical to SlimInfer's Table 1, so those were evidently carried over rather than re-run. Per-dataset, SPTS's worst losses on LLaMA are HotpotQA 55.64 → 53.66 and GovReport 34.99 → 33.49; PassageCount improves 7.00 → 7.24.

**Component ablation — TTFT speedup on LLaMA (Table 2)**:

| | 8K | 16K | 24K | 32K |
|---|---|---|---|---|
| + PAP | 1.08× | 1.24× | 1.35× | 1.44× |
| + PAP + LTP | 1.26× | 1.70× | 1.98× | 2.15× |
| + PAP + LTP + MSDP | 1.35× | 1.91× | 2.26× | **2.46×** |

LTP (the FFN half) contributes more than PAP (the attention half) — 0.71× of the 1.46× gain at 32K — which is the paper's evidence that attention-only sparsification leaves the larger cost on the table.

**KV-cache memory with PAP, LLaMA (Table 3, GB)**: full cache 4.00 / 8.00 / 12.00 / 16.00 at 8K/16K/24K/32K; with PAP 2.81 / 4.13 / 5.38 / 6.63 — savings 29.7% / 48.4% / 55.2% / 58.6%. These absolute figures are 4× those SlimInfer reports for the *same* model (1.00 GB at 8K, 4.00 GB at 32K); the paper does not say how it counts (the ratio matches counting 32 rather than 8 KV heads, i.e. a GQA-expanded cache), so compare only the percentages.

**Selection-signal ablations (LLaMA, LongBench avg)**
- *Proxy FFN vs. attention-only FFN skipping* (40% skipped in all deep layers, Table 4): S^MHA alone 46.31 → conditioned score 46.85 (+0.54).
- *Delayed pruning* (Table 5): without 46.11 → with 47.80 (+1.69); largest gains on Count (4.32 → 7.24) and RepoBench-P (52.39 → 55.20), attributed to reduced score dilution on long inputs.
- *Proxy scale* (Table 6, FLOPs per token per projection): default (512, 192) = 855K; dropping the SVD, (512, /), costs 2097K; dropping the channel cut, (/, 192), costs 2900K and is the least stable across datasets. Rank r matters more than D_low.
- *Top-ρ saliency* (Table 7): 47.56 → 47.80.
- *Query length for PAP* (Table 8): 64 tokens 47.00, 4 tokens 47.53, **1 token 47.80** — the single last token wins, the opposite of SlimInfer's finding that a 4-token window beats 1. The authors note multi-token averaging hurts TREC (63.0 vs. 72.5) and Count.

**Where it breaks / what is missing**
- **Irreversible after prefill.** A token skipped at layer l has no KV at layer l for the rest of generation; a stage-pruned token has none anywhere after that stage. Revival exists only across *layers* within prefill. Tasks where the decode trajectory changes what matters — the case LazyLLM's per-step re-selection was built for — are not separately tested.
- **"Training-free" needs an asterisk.** LTP requires an offline calibration pass and per-layer SVD, and the 200 calibration sequences are drawn from LongBench's Qasper subset — a slice of the evaluation benchmark.
- **Fixed budgets in absolute tokens** mean short prompts get little skipping (M = min(N, M_fixed)); speedup is 1.35× at 8K and the Table 1 ratios (1.36-1.68×) are LongBench-prompt averages. The headline 2.46× / 2.29× lives in Fig. 4 and Table 2 only.
- **Accuracy margin is thin**: +0.24 over SlimInfer on LLaMA, +0.07 on Qwen. The defensible claim is matched accuracy with better *decode* speed, and the decode comparison against SlimInfer/LazyLLM is figure-only (Fig. 4).
- Single-sequence latency on one A800; no batching, no serving-system integration; 1B-8B models only; code unreleased at v2.

**Abstract vs. body** — "up to 2.46× prefill and 2.29× end-to-end" is supported (Table 2 / Fig. 4). "State-of-the-art accuracy" holds by 0.07-1.13 points. The abstract's "training-free" is accurate in the gradient sense but the method is not calibration-free.

## Related

- [[lazyllm-dynamic-token-pruning-fu-2024]] and [[sliminfer-dynamic-token-pruning-long-2025]] — the dynamic-with-revival methods SPTS positions itself against; it argues their per-decode-step recompute of revived tokens caps end-to-end speed and that their previous-layer scores are "outdated."
- [[discovering-the-gems-in-early-layers-shi-2024]] — the static prune-once method, re-evaluated here at 47.09 / 1.31× on LLaMA and criticised for the extra forward pass; SPTS's Fig. 5b is the "prefilling-phase pruning" scheme GemFilter exemplifies.
- [[token-pruning]] (no page yet) — SPTS is the *skipping* variant: identity shortcut instead of deletion, plus delayed pruning at stage ends.
- [[residual-connection]] — the mechanism that makes skipping possible: Y = X + F(X), so an unselected token is exactly preserved.
- [[kv-cache]] — PAP produces a layer-ragged cache holding only active tokens per layer; this is where the decode-phase speedup comes from.
- [[attention-mechanism]] — the probe is the current layer's own last-token softmax attention; contrast [[speculative-prefill]] (no concept page yet; see [[speculative-prefill-liu-2025]]), which delegates selection to a small draft model.
- [[flash-attention]] — the dense baseline; all methods run on top of it.
- [[longbench]] (no page yet) — all 17 datasets; efficiency measured on 8K-32K samples drawn from it (20 per length).
- [[long-context-llm]] — the regime targeted; gains are minimal below 8K.
- [[llm-inference-roofline-yuan-2024]] — why FFN skipping (LTP) contributes more than attention skipping (PAP) to prefill time.
- [[llama]], [[qwen]] — LLaMA 3.1 8B Instruct and Qwen2.5 7B Instruct; openPangu-Embedded-1B on Ascend NPU is the third model.
- [[minference-dynamic-sparse-attention-jiang-2024]] — sparse-attention baseline re-measured here at 0.47× (LLaMA) and 0.41× (Qwen) TTFT on LongBench-length prompts, i.e. slower than dense FlashAttention.
- [[methods-to-speed-up-prompt-processing]] — where this belongs in the wiki's taxonomy of prefill-acceleration techniques.

## References

- [arXiv:2601.13155](https://arxiv.org/abs/2601.13155)
- FTP (training-free FFN token skipping): Xu et al., 2025 · PyramidInfer: Yang et al., 2024 · MInference: Jiang et al., NeurIPS 2024 · FlexPrefill: Lai et al., ICLR 2025
- LazyLLM is cited here as "Workshop on Efficient Systems for Foundation Models II @ ICML, 2024"
