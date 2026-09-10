---
title: "STS: Efficient Sparse Attention with Speculative Token Sparsity"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2605.15508"
authors:
  - Jiangnan Yu
  - Ceyu Xu
  - Yongji Wu
  - Yuan Xie
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# STS: Efficient Sparse Attention with Speculative Token Sparsity

**Jiangnan Yu\*, Ceyu Xu\*, Yongji Wu, Yuan Xie** (\*equal contribution; Yu, Xu, Xie at **HKUST**, Wu at [[uc-berkeley|UC Berkeley]]; Xie corresponding) — [arXiv:2605.15508](https://arxiv.org/abs/2605.15508), cs.LG / cs.CL, v1 15 May 2026, v2 18 May, **v3 21 Aug 2026** (read here), 14 pages / 12 figures. No venue stated. **No code release** mentioned anywhere in the paper (implementation is ~7,000 lines of Python + 2,400 of CUDA/C++ on [[vllm]] v0.9.2 and FlashInfer v0.1.6).

> [!important] Phase verdict — this is a DECODE-side method; file under [[sparse-attention]], not [[sparse-prefill]]
> STS runs inside a [[speculative-decoding]] loop: every speculative round the **draft model (Llama-3.2-1B) does a dense forward over the context**, its per-head attention scores are top-k'd into a **token-level, per-head mask**, the mask is remapped to the target's heads by an offline head-mapping table, and the **target (Llama-3.1-8B) verifies the draft's candidate tokens with sparse attention** over only the selected KV entries. All tokens keep their KV — nothing is evicted. The paper *claims* the same logic works for prefill (Table 1 marks "Sparse Prefill: Y"; § 4.2 describes a 2-D per-head mask), and it argues at length that decode-only sparsity breaks multi-turn agentic KV reuse. But **every speed number is a generation-throughput number** measured against decode-only baselines (Quest, TidalDecode) with a per-query *token budget*; the **default accuracy configuration "STS-D" explicitly keeps prefill dense**; the sparse-prefill mode "STS-P/STS-PD" appears only as a "stricter … stress test" line on two LongBench figures; and **no TTFT or prefill latency is reported anywhere**. It is the same shape as [[language-models-can-control-their-own-attention-ho-2026|Declarative Attention]] — a decode-time selector for a lossless KV cache — with a draft model instead of the model's own chain-of-thought doing the selecting.

## Problem

The paper's framing is **agentic serving**: long initial prefills, "prefill-after-decode" of tool output, multi-turn sessions whose KV caches persist and often must be offloaded to CPU while the agent waits on the environment. It sorts attention approximations into three bins (Fig. 1, Table 1):

- **Linear attention** (Linformer, Performer, Reformer, Mamba) — lossy, needs retraining.
- **[[token-pruning|Token pruning]] / KV eviction** (H2O, Scissorhands, StreamingLLM, SnapKV, PyramidInfer) — lossy and *irreversible*; "catastrophic for agentic workflows" because a token irrelevant now may be needed later, and the long tail of the attention distribution is where that information lives.
- **Token sparsity** (Quest, TidalDecode, NSA) — keep the full [[kv-cache]], attend to a dynamically chosen subset per query. This is the bin STS is in.

Against the existing token-sparsity methods it names three defects: (1) heuristic selectors miss critical tokens and so must keep many more to hold accuracy; (2) block-level selection caps sparsity; (3) selection is **just-in-time** on the target's critical path (Quest reads per-page min/max metadata from HBM during the target's own forward), so selection overhead eats the win. Plus one it stresses hardest: **Quest and TidalDecode support sparsity only in decode**, so in a multi-turn loop the sparsely-decoded turn *N* must be densely re-prefilled in turn *N+1*, "effectively break[ing] inter-turn KV caching". (STS's own headline configuration, as noted above, also prefills densely.)

## Mechanism

**Core observation.** The top-k attention indices of a small draft model correlate strongly with those of a larger target of the same family — "small language models share the same fundamental attention topology as large models" (Fig. 12 shows the draft-derived mask tracking an oracle within a few points at every layer).

**1. Offline head mapping (Algorithm 1).** Draft and target differ in width and depth, but every attention head's score matrix is N×N regardless of model size, so the mismatch reduces to *which draft head predicts which target head*. Both models run a prefill-only pass over the **first 100 samples of WikiText-2**; for each target head, the draft head whose top-k index set has the **largest intersection** with the target's top-k set is chosen (deliberately *not* Pearson correlation — "the ideal metric must evaluate the similarity of the resulting token masks, not the raw attention values"). It is a **one-to-many** map (one draft head serves several target heads) and it is **specific to each k**. No training, no online cost.

**2. Online mask generation (§ 4.2).** Dense draft forward → extract per-head attention weights → **top-k per draft head** → remap to target heads → target attends only to the indexed KV entries. Selection is at **token granularity**; a block variant applies top-k over pages instead, to suit paged-KV kernels (the `p_s` page-size knob: `p_s = 1` token-level, `p_s = 8` block-level). For decode the mask is 1-D (one query, k keys); for prefill the paper says it would be 2-D per head.

**3. Coupling to speculative decoding (§ 4.3).** Per round, the draft "runs over the speculated context and emits the attention logits"; the controller turns them into masks; the target's **verification pass** consumes them. Accepted tokens: their KV persists, nothing to do. Rejected bundle: the mask is discarded with it, and the next draft pass makes a fresh mask. Masks are made exactly once per speculative batch. Because the draft runs *ahead* of the target, the mask is **"known in advance"** — which is what enables the two system tricks below.

**4. Implementation (§ 5).** FlashInfer kernels extended to arbitrary per-head block-sparsity patterns (each head → its own KV block set and thread block; split-K for long sequences; KV blocks gathered into contiguous shared-memory tiles). The dense kernel is modified to dump QKᵀ into a global buffer so the draft's scores can be read. **Pipelining:** the draft's layer-*i* forward overlaps, on a separate CUDA stream, with mask generation for the target heads mapped to draft layer *i−1*; CUDA-graph compatible.

## Setup

| | |
|---|---|
| Draft → target | **Llama-3.2-1B-Instruct → Llama-3.1-8B-Instruct** (speed + accuracy); **Llama-3.1-70B** for accuracy only (Figs. 7, 9) |
| Hardware | **NVIDIA H20** (96 GB HBM3), PCIe Gen5 ×16 (128 GB/s bidirectional) for the offload study |
| Stack | vLLM 0.9.2 + FlashInfer 0.1.6; greedy decoding |
| Baselines | Dense (full attention), Quest, TidalDecode, StreamingLLM |
| Budgets | per-query token budget 256–4096 (accuracy), 512–4096 (speed); 4096 used for the "real-world" speed runs |

The **Dense baseline is described only as "Full Attention"**; the paper never states whether it runs the same speculative-decoding loop. See the qualification below.

## Results

### Speed (Fig. 4; generation speedup vs Dense, Llama-3.1-8B, H20)

| Setting | STS | TidalDecode | Quest |
|---|---|---|---|
| 10K context, budget 512 → 4096 | **1.36×** → ≈1.2× | ≈1.15× → ≈1.1× | ≈0.95× (below dense) |
| 32K, budget 512 → 4096 | **2.06×** → ≈1.5× | ≈1.7× → ≈1.3× | ≈1.25× → ≈1.15× |
| 100K, budget 512 → 4096 | **2.88×** → ≈2.2× | ≈2.25× → ≈2.05× | ≈1.85× → ≈1.75× |
| LongBench NarrativeQA (≈33.9K), budget 4096 | **≈2.67×** | ≈2.4× | ≈2.1× |
| HotpotQA (≈13.3K), 4096 | ≈2.65× | ≈2.25× | ≈1.4× |
| TriviaQA (≈10.9K), 4096 | ≈2.4× | ≈2.15× | ≈1.4× |
| GovReport (≈8.8K), 4096 | ≈1.9× | ≈1.7× | ≈1.3× |
| MultiFieldQA (≈6.9K), 4096 | ≈1.8× | ≈1.6× | ≈1.25× |
| Qasper (≈6.2K), 4096 | ≈1.6× | ≈1.45× | ≈1.15× |
| [[swe-bench\|SWE-bench]] trace (100 instances) | ≈1.35× | ≈1.2× | ≈1.05× |

Bold values are stated in the text; the rest are read off the bars. **The abstract's "2.67× at ~90% sparsity on NarrativeQA" checks only as a bar:** it is printed nowhere in the body; NarrativeQA's mean length is ≈33,855 tokens (Fig. 8 caption) and the budget is 4,096, so 4096/33855 = 12.1% kept ≈ **88% sparsity**, which the abstract rounds to 90%. Sequence lengths and budgets come from different figures; the join is ours.

> [!warning] The speedup's composition is not separated
> Qasper prompts average 6.2K tokens; with a 4,096 budget only ~⅓ of the KV is skipped, yet STS reports ≈1.6× and GovReport (8.8K, ~53% skipped) ≈1.9×. Attention sparsity alone cannot produce that on an 8B model whose per-token decode cost is dominated by weights at these lengths. Either the "Dense" baseline runs *without* speculative decoding (so part of the win is ordinary spec-decode acceptance), or the baseline is vLLM's default kernel path rather than the paper's own FlashInfer build. The paper does not say. Treat the absolute speedups as *STS-pipeline vs. some dense pipeline* and the **relative ordering vs Quest/TidalDecode** — which do share the sparse kernels — as the reliable signal.

### KV-offload prefetch (Fig. 5; 1K–512K context, sparsity 1/8, 1/16, 1/32)

Because the mask exists before the target's layers run, the needed KV blocks can be prefetched from CPU RAM over PCIe during draft/target compute. On-demand offload runs **14–19× slower** than GPU-resident; STS prefetch cuts that to **6–8×** — the paper calls it "roughly 3×", which is generous (14/6 ≈ 2.3×, 19/8 ≈ 2.4×). It is still 6–8× slower than keeping the KV on the GPU.

### Draft overhead (Fig. 6)

Llama-3.2-1B serves **>6,000 tok/s at concurrency 128** (≈7,500 in the figure) against ≈4,000 for the 8B and ≈1,900 for the 70B target, scaling linearly from concurrency 8 to 128 — so one draft can feed several targets. Note the draft still does *dense* attention over the whole context every round; for a 1B→8B pair that puts a floor of roughly ⅛ of the target's dense attention cost under any sparsity level.

### Accuracy

| Probe | Result |
|---|---|
| Zero-shot ARC-C, GSM8K, HellaSwag, [[mmlu\|MMLU]], TruthfulQA, Winogrande (Fig. 7) | within **0.5%** of dense for both 8B and 70B |
| [[longbench\|LongBench]] 6 tasks, Llama-3.1-8B (dense avg 51.2), budget 4096, `p_s=1` decode-sparse (Fig. 8) | ~1–1.5 F1 below dense per task (NarrativeQA ≈25.5 vs ≈26.5; Qasper ≈16 vs ≈17.5; TriviaQA ≈76 vs ≈77); Quest ≈13 and TidalDecode ≈18 on NarrativeQA at the same budget |
| LongBench, Llama-3.1-70B (dense avg 60.7), budget 4096 (Fig. 9) | same pattern; GovReport ≈60 vs ≈62, NarrativeQA ≈26 vs ≈29 |
| Block-level `p_s=8` vs token-level `p_s=1` | "nearly 98%" of the fine-grained score retained |
| **Sparse-prefill mode STS-PD** (`p_s=8`, dotted line) | ≈1–3 F1 below STS-D on most tasks; "performs comparably" per the text |
| Wiki-103 perplexity to 10K (Fig. 10) | STS within **0.01** of dense (≈6.38 vs ≈6.36 at 10K); Quest/TidalDecode ≈6.43, TidalDecode drifting (Δ > 0.5 claimed beyond 8K — the figure shows ≈0.07) |
| Passkey top-10 recall, 10K context (Fig. 11) | STS **≈0.905 at budget 64 → ≈0.955 at 1024**; Quest 0.85 → 0.94; TidalDecode 0.70 → 0.93 |

At low budgets (256–512) STS degrades far more gracefully than the heuristics — HotpotQA-8B at 256: STS ≈13 vs Quest ≈10 vs TidalDecode ≈11, dense 22.

### Layer-wise sparsity (Fig. 12, Llama-3.1-8B, oracle = max prunable at ΔPPL < 0.01)

An inverted-U: layers 1–2 tolerate only **5–10% sparsity** (the [[attention-sink]] / local-feature layers), layers 3–28 tolerate **85–95%**, and the last few layers drop back to ~70–75% (retain ~30%). The draft-guided mask tracks the oracle within a few points at every layer — the paper's evidence that cross-scale head correlation is real, and the reason a *uniform* budget is wasteful in the first two layers.

## Claims that did not check out, or need qualification

- **">98% recall with a budget of just 256 tokens out of 10K"** (§ 6.3.2) — **does not match Fig. 11**, where STS reads ≈0.93 at 256 and never exceeds ≈0.955 even at 1,024. The Fig. 11 caption repeats ">98%". Either the figure is wrong or the text is; take the figure.
- **"Perplexity within 0.01"** — consistent with Fig. 10 at 10K; but "TidalDecode Δ > 0.5 beyond 8K" is not — the figure shows ≈0.05–0.07.
- **"Roughly 3× reduction"** in offload latency — 14–19× → 6–8× is ≈2.3–2.4×.
- **"2.67× at ~90% sparsity on NarrativeQA"** — matches a Fig. 4(b) bar at ≈88% sparsity (derived), not a stated number.
- **"Applicable to both prefill and decode"** — architecturally plausible but *never timed*; the only prefill evidence is the STS-PD accuracy line. And that line's description — "the prompt itself is compressed on-the-fly, retaining only critical blocks in memory … non-essential tokens are pruned during prompt processing" — describes **KV dropping**, which contradicts § 4.2's 2-D-mask account and the paper's own "lossless, full KV cache" thesis. The paper never reconciles the two, and names the mode both "STS-P" and "STS-PD".
- **Speedup composition** vs the Dense baseline — undefined (callout above).
- Figure references in § 6.2.2–6.2.3 point to "Figure 7" for both the offload latency and the draft-throughput plots in the HTML version (the PDF has Figs. 5 and 6 correctly).
- Verified: author list and affiliations, v1–v3 dates, 1.36× / 2.06× / 2.88× at 10K / 32K / 100K (budget 512), budget 4096 for real-world runs, H20 / PCIe Gen5, vLLM 0.9.2 / FlashInfer 0.1.6 / 7,000 + 2,400 LoC, 100 WikiText-2 mapping samples, 14–19× vs 6–8×, >6,000 tok/s draft at 128 concurrency, 0.5% on general benchmarks, dense LongBench averages 51.2 / 60.7, ≈98% for `p_s=8`, layer-wise 5–10% / 85–95% / ~30%-retained.

## Limitations

- **Decode-side only, in practice.** No prefill/TTFT measurement; the sparse-prefill mode is an accuracy stress test with an internally inconsistent description. Everything that makes this attractive for "prefill-bound agentic workloads" is asserted, not shown.
- **Needs a same-family draft** with a compatible tokenizer and the offline head map; the map is per-k, so changing budget means re-mapping. Cross-family drafts are not tested (compare [[cross-family-speculative-prefill-upasani-2026]] on the prefill side).
- **The draft is dense.** Its full-context attention every round bounds the achievable saving at long context and is not counted as "selection overhead" in the paper's framing.
- **Single GPU type (H20), one target family, batch-1-style latency for the scaling plot**; the concurrency study covers the draft alone.
- **No code**, and the baseline pipeline is under-specified, so the absolute speedups are hard to reproduce or compare across papers.
- The whole method inherits [[speculative-decoding]]'s failure mode: when the draft's proposals are rejected, the mask is discarded and regenerated — at low acceptance rates the draft's dense passes become pure overhead.

## Related

- [[sparse-attention]] — where this files: a decode-time selector over a lossless [[kv-cache]], next to Quest/TidalDecode and [[language-models-can-control-their-own-attention-ho-2026|Declarative Attention]]. The wiki already caught one decode-side method being mis-filed as prefill; STS is the second candidate for that mistake, and its own abstract invites it.
- [[sparse-prefill]] — **not** this. The families there shrink or sparsify *prompt processing*; STS's measured wins are in generation.
- [[speculative-decoding]] — the host loop; STS is the first source in the wiki that gives the draft a second job (mask oracle) inside the *same* loop.
- [[speculative-prefill]] · [[speculative-prefill-liu-2025]] · [[omlx-specprefill-github-2026]] — the prefill-side cousins: there the draft scores prompt tokens *once* and the target prefills only the survivors, **dropping their KV** and shrinking the FFN too; here the draft re-scores every round and the target keeps every KV entry. STS-PD, as described, drifts toward the former.
- [[specattn-speculating-sparse-attention-shah-2025]] — the earlier (Oct 2025), single-author version of the same idea: draft attention → target KV selection during speculative decoding, with KL-divergence layer alignment and top-p instead of head mapping and top-k. STS does not cite it.
- [[sale-low-bit-estimation-sparse-attention-prefill-ji-2025]] — the prefill-side estimator ingested alongside: 4-bit QK estimation instead of a draft model, kernel-level, no spec-decoding dependency. Complementary phases.
- [[token-pruning]] · [[lazyllm-dynamic-token-pruning-fu-2024]] — the "lossy" bin STS argues against; LazyLLM is the JIT foil in § 7.3.
- [[minference-dynamic-sparse-attention-jiang-2024]] — filed by STS as "static"; a prefill method.
- [[prefix-caching]] · [[llm-agent]] — the multi-turn KV-reuse argument that motivates the (unmeasured) prefill mode.
- [[attention-sink]] — the explanation offered for dense layers 1–2.
- [[arithmetic-intensity]] · [[long-context-llm]] · [[methods-to-speed-up-prompt-processing]] — decode at long context is KV-bandwidth-bound, which is why a token budget translates to wall-clock here in a way it would not for prefill.
- [[vllm]] · [[llama]] · [[nvidia]] · [[uc-berkeley]] · [[longbench]] · [[swe-bench]] · [[mmlu]]
- [[needle-in-a-haystack]] — **no page yet**; the passkey probe.

## References

- [arXiv:2605.15508](https://arxiv.org/abs/2605.15508) (v3, 21 Aug 2026)
- Quest: Tang et al., ICML 2024, [arXiv:2406.10774](https://arxiv.org/abs/2406.10774) — query-aware page selection at decode; main baseline.
- TidalDecode: Yang et al., [arXiv:2410.05076](https://arxiv.org/abs/2410.05076) — early-layer scores select tokens for later layers; baseline.
- StreamingLLM: Xiao et al., [arXiv:2309.17453](https://arxiv.org/abs/2309.17453) — static sink + window baseline.
- Native Sparse Attention: Yuan et al. (DeepSeek), [arXiv:2502.11089](https://arxiv.org/abs/2502.11089) — the trained-gate alternative.
- SeerAttention: Gao et al., [arXiv:2410.13276](https://arxiv.org/abs/2410.13276) — learned block-sparsity gate, prefill-focused.
- Speculative decoding: Leviathan et al., [arXiv:2211.17192](https://arxiv.org/abs/2211.17192); Chen et al., [arXiv:2302.01318](https://arxiv.org/abs/2302.01318).
- FlashInfer: Ye et al., [arXiv:2501.01005](https://arxiv.org/abs/2501.01005); vLLM: Kwon et al., SOSP 2023 — [[vllm]].
- LazyLLM: Fu et al., [arXiv:2407.14057](https://arxiv.org/abs/2407.14057) — [[lazyllm-dynamic-token-pruning-fu-2024]].
- InfLLM: Xiao et al., [arXiv:2402.04617](https://arxiv.org/abs/2402.04617) — CPU-offloaded KV blocks; the offload comparison point.
