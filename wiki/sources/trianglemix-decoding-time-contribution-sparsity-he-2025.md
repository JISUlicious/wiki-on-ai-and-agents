---
title: "TriangleMix: Accelerating Prefilling via Decoding-time Contribution Sparsity"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2507.21526"
authors:
  - Zhiyuan He
  - Yike Zhang
  - Chengruidong Zhang
  - Huiqiang Jiang
  - Yuqing Yang
  - Lili Qiu
year: 2025
status: complete
importance: high
tags:
  - 2025
  - paper
---

# TriangleMix: Accelerating Prefilling via Decoding-time Contribution Sparsity

**Zhiyuan He, Yike Zhang, Chengruidong Zhang, Huiqiang Jiang, Yuqing Yang, Lili Qiu** ([[microsoft-research|Microsoft Research]]; Yike Zhang at [[tsinghua-university|Tsinghua]], work done during a Microsoft internship) — [arXiv:2507.21526](https://arxiv.org/abs/2507.21526), cs.CL. **Four versions with two title changes:** v1 (29 Jul 2025) *"TriangleMix: A Lossless and Efficient Attention Pattern for Long Context Prefilling"*; v2 (11 Oct 2025) *"TriangleMix: Accelerating Prefilling via Decoding-time Contribution Sparsity"*; v3/v4 (20–21 Apr 2026) drop the "TriangleMix:" prefix and read *"Accelerating Prefilling via Decoding-time Contribution Sparsity"*. This page follows **v4**. **No venue stated** in any version. **Code:** `aka.ms/TriangleMix` → [github.com/microsoft/MInference/tree/main/TriangleMix](https://github.com/microsoft/MInference/tree/main/TriangleMix).

**Same group as MInference — confirmed.** Jiang, C. Zhang, Yang and Qiu are four of the twelve authors of [[minference-dynamic-sparse-attention-jiang-2024|MInference]]; the code lives inside the MInference repository; and the paper is written as a critique-from-within of the dynamic-sparse line (MInference, FlexPrefill, XAttention) that this group started.

> [!important] What this is, in one paragraph
> A **static [[sparse-attention]] pattern** chosen by a **gradient probe** rather than by attention magnitude. The finding: split the causal attention matrix into a *Streaming* section (sink columns + local band), a *Last Q–K* section (the bottom rows, i.e. the last queries against every key) and a *Middle Q–K* section (everything else). Middle Q–K carries attention scores as large as Last Q–K's — so dynamic-sparse estimators keep computing it — yet its **gradient with respect to the first decoded token's logit is near zero in half the layers** of Llama-3-8B. Dropping it in those layers ("Triangle attention", O(N)) is nearly lossless on RULER / LongBench / GSM-Infinite, and because no index estimation is needed the kernel is **15.3× faster than FlashAttention at 128K** (dynamic sparse: 1.9–3.4×). **But that 15.3× is a per-layer kernel number in the 16 of 32 Triangle layers; the end-to-end TTFT gain on Llama-3.1-8B is −12 % at 32K to −32 % at 128K (1.14×–1.47×), because attention is only ~34 % / ~70 % of prefill at those lengths and half the layers stay dense.** Stacked on MInference it takes 128K TTFT from 18.0 s to 14.5 s. Training-free; the layers are chosen offline once per model.

## Problem

Dynamic sparse attention ([[minference-dynamic-sparse-attention-jiang-2024|MInference]], FlexPrefill, XAttention) rests on one intuition: skip the attention blocks with negligible scores, compute the ones with non-trivial scores. Its costs are the O(N²) index estimation and the complexity of the resulting kernels; its accuracy assumption is that a high-score block must be computed. The paper asks whether that assumption is true: *"Must we compute all non-trivial blocks during prefilling to preserve decoding accuracy?"* — and answers no, by exhibiting a second kind of sparsity it calls **decoding-time contribution sparsity**: blocks that have real attention mass during prefill but whose removal does not move the decoded tokens. Static patterns (StreamingLLM, LM-Infinite) already skip the interior of the matrix in every layer, but at a large accuracy cost at long context; the question is which layers can afford it.

## Mechanism

**Gradient probe (§ 2.1, App. A.3).** Multiply every attention weight by a probe variable θ_{l,i,j} = 1, run the forward pass, take the logit y_gt of the ground-truth first output token, and read off ∂y_gt/∂θ_{l,i,j}. The importance of a region M̂ at layer l is the mean gradient over the region, Grad(M̂, l). The probe task is a synthetic **key–value retrieval** over UUID pairs (~2,000-token prompts, 100 samples; the model must emit the value as its first token). Regions are defined by masks with sink size *si* = 64, window *sl* = 128 and *last* = 128 rows for the probe. Generalises to GQA.

**Findings (§ 2.2, Fig. 3–4).** Streaming has both high scores and high gradients (the known [[attention-sink]] result). Middle Q–K has *slightly higher* average attention scores than Last Q–K but *substantially lower* gradients — and its gradient is strongly **layer-wise sparse**: on Llama-3.1-8B-Instruct and Llama-3-8B-262K "the gradients of the last 16 layers are near zero". The interpretation, **locality of influence**: an attention score at query position *t* affects predictions only up to about *t* + Δt (Δt ≈ 128), so the interior of the matrix influences intermediate prompt tokens but not the tokens generated after the prompt. Because the current Middle Q–K is a subset of every later token's Middle Q–K, the argument extends beyond the first decoded token.

**Perplexity check (§ 2.3, Table 1).** 1,024 Wikipedia articles, 2,048 tokens each; skip attention for Q indices 512–1,024 (keeping Streaming). At positions 1,024–1,152 — where that skipped region *is* the Last Q–K — perplexity rises **+0.27 / +0.28 / +0.11** even when only the gradient-lowest half of the layers skip it, and +0.45 / +0.49 / +0.49 when all layers do. At positions 1,920–2,048 — where the skipped region has become Middle Q–K — skipping it in half the layers costs only **+0.01 / +0.01 / +0.02** (Llama-3.1 / Llama-3-262K / Qwen2.5-7B), and skipping it in all layers +0.09 / +0.10 / +0.10.

**The Triangle pattern (§ 2.4, App. A.1) — precise definition.** For a prompt of N tokens, a Triangle layer keeps exactly two things:

| Kept | Rows | Columns | Cost |
|---|---|---|---|
| **Streaming section** | all *i* < N − N_last | the first N_sink = **8** keys, plus the band *i* − *j* ≤ N_window = **512** | O(N · 520) |
| **Last Q–K section** | the last **N_last** rows | *all* keys *j* ≤ *i* (full causal attention) | O(N_last · N) |

and skips the **Middle Q–K** — rows *i* < N − N_last, columns 8 < *j* < *i* − 512 — the triangle the name refers to. Total O(N). The kernel (Algorithm 1, Triton) runs streaming attention for the upper rows and a split-K chunked FlashAttention for the last rows, merging the partial outputs by their log-sum-exp. **N_last for the evaluation runs is not stated** (the probe used 128; only sink = 8 and window = 512 are given for evaluation). Since the last rows attend to every key, **K and V are still projected and written for all N tokens** — decode is untouched and the [[kv-cache]] is full-size; this is [[sparse-attention]] in the wiki's taxonomy, with no FFN saving.

**Which layers stay dense (§ 2.4, § 3.1, § 3.3).** Run the probe once per model, rank layers by Grad(M_middle, l), convert the **L_tri** lowest-contribution layers to Triangle, leave the rest dense (or hand them to a dynamic-sparse method). Defaults: **L_tri = 16 of 32** for Llama-3.1-8B-Instruct and Llama-3-8B-Instruct-262K, **L_tri = 8 of 28** for Qwen2.5-7B-Instruct. The 64K RULER sweep (Fig. 6, step 2) says L_tri = 20 still retains 99.7 % / 100.0 % on the two Llamas (a 62.5 % layer-sparsity ratio) and L_tri = 8 retains 98.2 % on Qwen (28.6 %); the defaults are chosen conservatively below those. v1 of the paper simply used "dense in shallow layers, Triangle in deep layers"; the gradient-ranked selection arrived in v2 — for the Llamas the two coincide, since the near-zero layers are the last 16.

## Setup

| | |
|---|---|
| Hardware | **single [[nvidia\|NVIDIA]] A100 80 GB**, batch 1 |
| Kernels | Triangle in Triton; dense baseline = FlashAttention (mis-cited to Kwon et al. 2023, the PagedAttention paper) |
| Models | Llama-3.1-8B-Instruct, Llama-3-8B-Instruct-262K (GradientAI), Qwen2.5-7B-Instruct |
| Benchmarks | RULER 4K–128K; all English [[longbench\|LongBench]] tasks; GSM-Infinite hard subset at 8K/16K/32K × 2/4/6/8 ops, 500 problems each (6,000) |
| Static baselines | Streaming (all layers), Triangle (all layers), StreamingMix (TriangleMix with Streaming in place of Triangle), DuoAttention at 50 % (official Llama-3.1 pattern; trained for Llama-3-262K; **did not converge on Qwen2.5**) |
| Dynamic baselines | MInference, FlexPrefill (γ = 0.95), XAttention (τ = 0.95, stride 8), each alone and with TriangleMix in the dense layers |

## Results

### Accuracy — "nearly lossless", and where it is not

RULER averages over 4K–128K (Tables 2–3):

| | Dense | **TriangleMix** | Δ | MInference | FlexPrefill | XAttention | DuoAttention | Triangle all layers | Streaming all layers |
|---|---|---|---|---|---|---|---|---|---|
| Llama-3.1-8B-Instruct | 90.4 | **90.2** | −0.2 | 89.8 | 89.9 | 89.5 | 84.7 | 72.7 | 36.5 |
| Llama-3-8B-262K | 86.5 | **86.6** | +0.1 | 86.8 | 84.3 | 86.5 | 86.0 | 74.1 | 33.3 |
| Qwen2.5-7B-Instruct | 86.0 | **85.6** | −0.4 | 85.7 | 78.1 | 84.7 | — | 61.4 | 36.5 |

The text's "−0.4 % to +0.1 %" checks (they are absolute points, not percent). Per length the worst cells are Qwen at 64K (81.9 → 80.2) and Llama-3.1 at 128K (78.1 → 77.5); DuoAttention falls apart at 64K–128K on Llama-3.1 (82.5 / 64.0 vs 86.3 / 77.5). Triangle in *every* layer costs 14–25 points — the Middle Q–K matters in the shallow layers — and Streaming everywhere is catastrophic.

LongBench averages (Tables 7–8): 54.8 → **54.5** (Llama-3.1), 44.2 → **43.4** (Llama-3-262K), 46.8 → **46.1** (Qwen2.5). GSM-Infinite (Tables 4, 9): 0.076 → 0.074, 0.060 → 0.060, 0.161 → 0.169 — the stated "−0.002 to +0.008" checks.

> [!warning] "Nearly lossless" hides a ~10-point PassageRetrieval loss on two of three models
> LongBench **PassageRetrieval**: Llama-3-262K **95.0 → 85.3**, Qwen2.5 **99.3 → 89.3**, Llama-3.1 99.7 → 98.3. Every other task moves by ≤ 1.3 points, so the averages barely register it. The paper's own diagnosis of the all-Triangle baseline — "attention over the Middle Q–K section in certain layers (especially shallow layers) is still crucial for retrieval tasks" — evidently still bites at L_tri = 16 / 8 for the exact task the probe was designed on. TREC (in-context learning) is where *StreamingMix* fails instead (71.0 → 51.0 on Llama-3.1), which is the evidence that the **Last Q–K rows in deep layers carry ICL**. Stacking is also not free on Qwen: TriangleMix + MInference is **63.5 at 128K vs 68.0 for MInference alone** (−4.5) and 76.9 vs 79.1 at 64K.

### Attention-kernel latency (Table 5, ms per layer, A100)

| | Llama-3.1-8B 32K | 64K | 128K | Qwen2.5-7B 32K | 64K | 128K |
|---|---|---|---|---|---|---|
| Dense (FlashAttention) | 44 | 179 | 750 | 40 | 156 | 646 |
| MInference | 94 (**0.5×**) | 152 (1.2×) | 229 (3.3×) | 93 (0.4×) | 151 (1.0×) | 245 (2.6×) |
| FlexPrefill | 32 (1.4×) | 71 (2.5×) | 220 (3.4×) | 38 (1.0×) | 99 (1.6×) | 310 (2.1×) |
| XAttention | 47 (0.9×) | 127 (1.4×) | 391 (1.9×) | 44 (0.9×) | 128 (1.2×) | 391 (1.7×) |
| **Triangle** | 12 (3.7×) | 24 (7.5×) | **49 (15.3×)** | 11 (3.7×) | 21 (7.4×) | 76 (8.5×) |

15.3× at 128K and the dynamic range 1.9–3.4× check (Llama; on Qwen the dynamic methods are 1.7–2.6× and Triangle 8.5×). Note the left column: at 32K **MInference is 2× slower than dense** and XAttention 10 % slower — the [[sparse-prefill]] rule's "below ~64K the pattern search costs more than it saves", measured by the people who wrote MInference. Triangle has no such floor because it has no search.

### End-to-end TTFT (Table 6, Llama-3.1-8B-Instruct, seconds, A100)

| | 32K | 48K | 64K | 80K | 96K | 112K | 128K |
|---|---|---|---|---|---|---|---|
| Dense | 4.1 | 7.3 | 11.2 | 15.9 | 21.3 | 27.5 | 34.5 |
| **TriangleMix** | 3.6 (−12 %) | 5.9 (−19 %) | 8.6 (−23 %) | 11.7 (−26 %) | 15.2 (−29 %) | 19.1 (−31 %) | **23.4 (−32 %)** |
| DuoAttention | 3.6 (−13 %) | 5.8 (−20 %) | 8.7 (−22 %) | 11.7 (−26 %) | 15.5 (−27 %) | 19.1 (−31 %) | 23.7 (−31 %) |
| MInference | 5.5 (**+34 %**) | 7.8 (+7 %) | 10.1 (−10 %) | 12.3 (−23 %) | 13.4 (−37 %) | 15.9 (−42 %) | 18.0 (−48 %) |
| **+ TriangleMix** | 4.2 (+2 %) | 6.0 (−18 %) | 7.7 (−31 %) | 9.5 (−40 %) | 10.9 (−49 %) | 12.7 (−54 %) | **14.5 (−58 %)** |
| FlexPrefill | 3.6 (−12 %) | 5.5 (−25 %) | 7.7 (−31 %) | 9.9 (−38 %) | 12.3 (−42 %) | 15.0 (−45 %) | 17.8 (−48 %) |
| + TriangleMix | 3.4 (−17 %) | 5.2 (−29 %) | 7.2 (−36 %) | 9.2 (−42 %) | 11.3 (−47 %) | 13.6 (−51 %) | 16.1 (−53 %) |
| XAttention | 4.1 (−1 %) | 6.6 (−10 %) | 9.4 (−16 %) | 12.4 (−22 %) | 15.7 (−26 %) | 19.3 (−30 %) | 23.2 (−33 %) |
| + TriangleMix | 3.5 (−14 %) | 5.5 (−24 %) | 7.7 (−31 %) | 10.0 (−37 %) | 12.4 (−42 %) | 15.1 (−45 %) | 17.8 (−48 %) |

**So the end-to-end number is 1.14× at 32K, 1.30× at 64K, 1.47× at 128K** — and the abstract, introduction and conclusion of v2–v4 quote only the 15.3× kernel figure; the "12 %–32 % TTFT" appears in § 3.4 (and was in v1's abstract). TTFT is reported for Llama-3.1-8B only. Best stack: TriangleMix + FlexPrefill for 32K–80K, TriangleMix + MInference for 96K–128K.

### The attention-share rule, derived from the paper's own tables

Table 5 is per layer; 32 layers × dense kernel time versus Table 6's dense TTFT gives attention's share of prefill for an 8B model on an A100: **32K: 1.41 s / 4.1 s = 34 %; 64K: 5.73 / 11.2 = 51 %; 128K: 24.0 / 34.5 = 70 %** — matching [[sparse-prefill]]'s "~¼ at 10K, ~70–80 % at 128K, > 90 % past 300K". And the arithmetic closes: 16 Triangle layers × (750 − 49) ms = 11.2 s saved at 128K → predicted 23.3 s vs measured 23.4 s; at 32K 16 × (44 − 12) = 0.51 s → 3.59 vs 3.6 s. Amdahl in one line: a 15.3× kernel in half the layers, with attention 70 % of the bill, can remove at most 0.5 × 0.70 × (1 − 1/15.3) = 33 % of TTFT. That is the ceiling the paper hits exactly, and it is why the only way past ~1.5× at 128K is either more Triangle layers (L_tri = 20 costs 0.3 % on RULER) or dropping tokens so the FFN shrinks too ([[token-pruning]], [[uniprefill-block-wise-dynamic-sparsification-fan-2026]]).

## Claims that did not check out, or need qualification

- **"Additional 6 %–19 % reduction in TTFT over dynamic sparsity alone"** — Table 6 gives **5–10 % on FlexPrefill, 15–23 % on XAttention, 19–24 % on MInference** (MInference at 32K: 5.5 → 4.2 s = −24 %; XAttention at 128K: 23.2 → 17.8 = −23 %). The stated range is neither the minimum-to-maximum nor the 128K column; the true span is **5–24 %**.
- **Headline is the kernel, not TTFT** — 15.3× applies to one layer's attention kernel in the 16 Triangle layers; end-to-end is 1.14–1.47× on Llama-3.1-8B, and there is no TTFT number at all for Qwen2.5 (where only 8/28 layers are Triangle, so it would be smaller).
- **"Nearly lossless"** — true for RULER/GSM-Infinite averages; PassageRetrieval drops ~10 points on two models, and TriangleMix + MInference loses 4.5 points at 128K on Qwen (see callout).
- **N_last is never given** for the evaluation configuration.
- **v1 described the pattern as "lossless"** in its title; v2 downgraded to "nearly lossless"; v3/v4 removed the name from the title. The numbers (15.3×, −12…−32 %, 18.0 → 14.5 s) are unchanged from v1.
- **FlashAttention cited as Kwon et al. 2023** (that is vLLM/PagedAttention).
- Verified: L_tri 16/16/8, sink 8 / window 512, γ = 0.95, τ = 0.95 stride 8, A100 80 GB, all Table 2–9 values quoted above, the 99.7 % / 100 % / 98.2 % retention and 62.5 % / 28.6 % ratios, the PPL deltas, +0.008 / −0.002 on GSM-Infinite.

## Limitations

- **Sparsity is a property of the checkpoint, not the method** (the paper's own limitation section): Qwen2.5-7B tolerates only 8 Triangle layers of 28, so its acceleration is bounded lower; the authors suggest post-training to induce more.
- **Attention-only**: KV for every token is still computed and stored; FFN, projections and decode are untouched, so the ceiling is attention's share of prefill (34 % at 32K, 70 % at 128K on an 8B model).
- **Layer choice is offline and task-specific**: the probe uses a synthetic KV-retrieval prompt at ~2K tokens with 100 samples; whether the same layers are right for summarisation or code at 128K is not tested beyond the benchmark averages, and the PassageRetrieval loss says the selection is not quite right even for retrieval.
- **Single A100, batch 1, three 7–8B models**; no serving-engine integration, no tensor-parallel results (though the paper notes TriangleMix avoids DuoAttention's per-head TP imbalance), no hybrid-architecture models.
- **The probe measures the first decoded token**; the extension to later tokens is an argument (subset property), not a measurement, apart from the 2K-token perplexity study.
- **N_last unspecified**; window 512 and sink 8 are shared with the static baselines, so the Streaming rows are not tuned against.

## Related

- [[sparse-attention]] · [[sparse-prefill]] — the family and the umbrella; this source is the cleanest quantitative statement of the attention-share ceiling (a 15.3× kernel → 1.47× TTFT) and the cleanest measurement of dynamic sparsity's short-context floor (MInference 0.5× at 32K).
- [[minference-dynamic-sparse-attention-jiang-2024]] — the same group's dynamic method, here used as both baseline and partner; its A-shape and vertical-slash patterns are magnitude-driven, whereas Triangle is contribution-driven. Vertical-slash's "last 64 queries" estimator and Triangle's Last Q–K section are the same intuition seen from two sides.
- [[uniprefill-block-wise-dynamic-sparsification-fan-2026]] — ingested alongside; scores tokens from the last 128 queries' attention and drops them from all sublayers. TriangleMix supplies the *why* for that selector (locality of influence: middle-row attention does not reach the decoded tokens), and UniPrefill supplies what TriangleMix cannot — FFN savings and hybrid-architecture coverage.
- [[token-sparse-attention-jo-2026]] · [[claa-cross-layer-attention-aggregation-mcdanel-2026]] — the other 2026 sources the wiki's attention-share numbers were derived from; TriangleMix's Tables 5–6 confirm them independently on an A100.
- [[token-pruning]] · [[lazyllm-dynamic-token-pruning-fu-2024]] · [[sliminfer-dynamic-token-pruning-long-2025]] · [[discovering-the-gems-in-early-layers-shi-2024]] — the route that *does* shrink the FFN; complementary, not competing (a Triangle layer could still run on a pruned sequence).
- [[speculative-prefill]] · [[draft-based-approximate-inference-galim-2025]] — SpecKV-PC composes token selection with vertical-slash attention; composing it with Triangle would be the obvious next stack.
- [[attention-sink]] · [[massive-activations-attention-sinks-sun-2026]] — the Streaming section's high gradient *and* high score is the sink phenomenon; StreamingLLM / LM-Infinite are the all-layers static baselines.
- [[flash-attention]] · [[flash-attention-dao-2022]] — the dense kernel the 15.3× is measured against; Algorithm 1 is a streaming + split-K FlashAttention hybrid.
- [[kv-cache]] · [[paged-attention]] — untouched here; the paper lists SnapKV and Quest as orthogonal.
- [[long-context-llm]] · [[longbench]] · [[ruler]] (**no page**) · [[grouped-query-attention]] — evaluation and applicability notes.
- [[mechanistic-interpretability]] — the gradient probe is an attribution method applied to attention edges; it is the only prefill-acceleration paper in the wiki that chooses its pattern by attribution rather than by magnitude.
- [[methods-to-speed-up-prompt-processing]] — the "shrink it" tier; the Table 6 numbers are a good reference row for an A100 at 128K (34.5 s dense).
- [[language-models-can-control-their-own-attention-ho-2026]] — a decode-time relative: sparsity declared by the model rather than found by probing.
- [[pop-prefill-only-pruning-he-2026]] — the same asymmetry one level up: POP finds deep layers' loss-sensitivity near zero when they only encode context, and skips those layers for prompt tokens; TriangleMix finds the deep layers' *interior attention* contributes nothing to decoding and skips that. Both are gradient/sensitivity-chosen, training-free, and cut prefill only.
- [[sale-low-bit-estimation-sparse-attention-prefill-ji-2025]] — a dynamic-sparse peer that attacks the estimator cost with INT4 Q·K rather than removing the estimator; its 24 % overhead at 8K is the same floor Table 5 shows for MInference at 32K.
- FlexPrefill, XAttention, DuoAttention — **no source pages**; cited below.

## References

- [arXiv:2507.21526](https://arxiv.org/abs/2507.21526) (v1 29 Jul 2025 → v4 21 Apr 2026) · code [github.com/microsoft/MInference/tree/main/TriangleMix](https://github.com/microsoft/MInference/tree/main/TriangleMix)
- MInference: Jiang et al., NeurIPS 2024, [arXiv:2407.02490](https://arxiv.org/abs/2407.02490) — [[minference-dynamic-sparse-attention-jiang-2024]].
- FlexPrefill: Lai et al., ICLR 2025, [arXiv:2502.20766](https://arxiv.org/abs/2502.20766). XAttention: Xu et al., [arXiv:2503.16428](https://arxiv.org/abs/2503.16428). DuoAttention: Xiao et al., ICLR 2025, [arXiv:2410.10819](https://arxiv.org/abs/2410.10819) — learned retrieval/streaming head split; the static baseline with the same TTFT and worse long-context accuracy.
- StreamingLLM: Xiao et al., [arXiv:2309.17453](https://arxiv.org/abs/2309.17453); LM-Infinite: Han et al., [arXiv:2308.16137](https://arxiv.org/abs/2308.16137) — the Streaming section.
- FlashAttention: Dao et al., [arXiv:2205.14135](https://arxiv.org/abs/2205.14135) — [[flash-attention-dao-2022]] (the paper cites Kwon et al. for it).
- RULER: Hsieh et al., [arXiv:2404.06654](https://arxiv.org/abs/2404.06654); LongBench: Bai et al., [arXiv:2308.14508](https://arxiv.org/abs/2308.14508); GSM-Infinite: Zhou et al., [arXiv:2502.05252](https://arxiv.org/abs/2502.05252).
- SnapKV: Li et al., [arXiv:2404.14469](https://arxiv.org/abs/2404.14469); Quest: Tang et al., [arXiv:2406.10774](https://arxiv.org/abs/2406.10774) — the orthogonal KV-side methods.
- Triton: Tillet et al., MAPL 2019.
