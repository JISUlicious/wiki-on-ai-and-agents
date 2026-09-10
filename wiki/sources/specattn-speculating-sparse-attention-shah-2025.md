---
title: "SpecAttn: Speculating Sparse Attention"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2510.27641"
authors:
  - Harsh Shah
year: 2025
venue: NeurIPS 2025 Workshop on Structured Probabilistic Inference & Generative Modeling
status: complete
importance: low
tags:
  - 2025
  - paper
---

# SpecAttn: Speculating Sparse Attention

**Harsh Shah** (Machine Learning Department, [[cmu|Carnegie Mellon University]]) — [arXiv:2510.27641](https://arxiv.org/abs/2510.27641), cs.CL / cs.LG / eess.SY, submitted 31 Oct 2025, v1 only. **Venue: NeurIPS 2025 Workshop on Structured Probabilistic Inference & Generative Modeling** (per the arXiv comments field). Single-author workshop paper; **no code release** stated.

> [!important] Phase verdict — DECODE only; file under [[sparse-attention]] next to Quest, not under [[sparse-prefill]]
> Algorithm 1, line 6: **"Prefill both models"** — draft and target both prefill the prompt densely and build full KV caches. Sparsity enters only in the *verification* pass of [[speculative-decoding]]: during each round the draft (TinyLlama-1.1B) generates γ candidate tokens and its attention rows are saved; a top-*p* nucleus over those rows, unioned across the γ steps and remapped layer-to-layer, becomes a binary mask; the target (Llama-2-7B) then verifies the γ candidates attending only to the masked KV entries. Nothing is evicted — the paper calls it "KV cache pruning", but it means *skipping reads*, not deleting entries. There is no prefill-side mechanism, no TTFT number, and the whole evaluation is at **≤2,048 tokens**, 90% of which are *generated*. This is the October-2025 precursor of [[sts-speculative-token-sparsity-yu-2026|STS]] (May 2026), which does not cite it.

## Problem

Dense attention over the KV cache is the per-token cost that grows with context during generation; static sparse architectures (Longformer, BigBird) need retraining; inference-time methods (MInference, SpargeAttn) "offer faster prompt processing" but their benefits "diminish on shorter contexts or during the token-by-token generation phase"; Quest and StreamingLLM select at decode but with heuristics or fixed patterns. Speculative decoding already runs a draft model that computes attention over the same context — the paper's claim is that those attention weights are a free importance signal for the target's decode attention, so the two techniques, normally treated as orthogonal, can share work.

## Mechanism

Three components, in order of the paper's contribution list.

**1. Layer mapping by KL divergence (offline, Appendix B).** Draft and target differ in depth (TinyLlama 22 layers → Llama-2-7B 32 layers). For each target layer *j* and draft layer *i*, similarity is S_{i,j} = −D_KL(A^v_j ‖ A^d_i) over attention distributions collected on WikiText. The assignment must be **monotonic** — as you go up the target, the matched draft layer never goes down — on the intuition that attention patterns evolve similarly with depth in both models. That makes it a dynamic-time-warping problem solved by DP (Algorithm 3: diagonal = match, left = reuse the same draft layer, above = skip draft layers). A draft layer may serve several target layers (Fig. 2 heatmap shows the chosen cells). Note that A ∈ ℝ^L is a **per-layer** distribution — heads are aggregated — unlike STS's per-head mapping.

**2. Sorting-free top-*p* nucleus selection (Algorithm 2, Triton).** For each draft step's attention row, binary-search a threshold θ such that the mass of entries ≥ θ reaches *p*·Σa (tolerance 10⁻⁶ in the pseudocode; **10 fixed iterations** in the implementation), take those indices, and **union across the γ speculative steps**. Top-*p* rather than top-*k* because a fixed budget is "inefficient for peaked distributions or insufficient for flat" ones; Appendix A gives the bound ‖O − Ô‖ ≤ (1 − p)·‖V‖, i.e. the retained attention mass directly bounds the output error. The union means the mask is shared by all γ verification queries. The idea is credited to Twilight (Lin et al. 2025).

**3. Sparse verification.** The mask is converted to **CSR** and fed to FlashInfer's `BlockSparseAttention`; chunk size 16 (to match Quest's page size). The first **2 target layers stay dense** for all methods, following Quest's observation of diffuse early-layer attention. After verification, accepted tokens are appended to both caches as usual.

Per round the draft therefore does dense attention over the full context γ times, and the target does one sparse pass over γ+1 queries.

## Setup

| | |
|---|---|
| Draft → target | **TinyLlama-1.1B** (intermediate-step-1431k-3T) → **Llama-2-7b-hf** |
| Hardware | **one RTX 4090** (24 GB) |
| Perplexity data | PG-19, truncated to **2,048 tokens**: first 10% (≈205) prefilled, remaining 90% scored as decode — "simulating long generation" |
| Throughput data | LongBench `gov_report`, same 10/90 split |
| Baselines | full attention; StreamingLLM; Quest (same ~77% KV reduction) |

## Results

### Perplexity vs KV reduction (Table 1, PG-19, Llama-2-7B)

| Method | PPL | ΔPPL | relative | KV reads skipped |
|---|---|---|---|---|
| Full attention | 6.435 | — | — | — |
| StreamingLLM | 186.242 | +179.807 | +2,794% | 77.4% |
| Quest | 7.823 | +1.389 | **+21.58%** | 77.4% |
| **SpecAttn p = 0.95** | **7.419** | **+0.984** | **+15.29%** | **78.4%** |
| SpecAttn p = 0.97 | 6.720 | +0.285 | +4.43% | 68.8% |
| SpecAttn p = 0.99 | 6.471 | +0.036 | +0.56% | 44.3% |

The abstract's "over 75% reduction in KV cache accesses with a mere 15.29% increase in perplexity" is the p = 0.95 row and checks (0.984 / 6.435 = 15.29%). At matched ~78% sparsity SpecAttn beats Quest by 6.3 relative points of perplexity; StreamingLLM's static window collapses on PG-19 because it discards everything in the middle. Appendix C (Fig. 5) shows Quest's cumulative perplexity diverging from the baseline faster than SpecAttn's as decoding proceeds.

### Kernel speed (Figs. 3–4)

- **Sorting-free selection:** 0.03 ms at KV 256–2,048, 0.04 ms at 4,096, 0.05 ms at 8,192, vs 0.19–0.24 ms for a PyTorch sort (with noisy 3.43 ms and 2.19 ms outliers at 256 and 4,096) — **≥4× faster up to 8,192** as stated; 5–7× at the clean points.
- **Sparse attention kernel:** FlashInfer BSR at p = 0.97 vs p = 1.0, averaged over layers: **≈3.3× at 512, ≈3.8× at 1,024, ≈4.2× at 2,048 tokens**, with a wide layer spread (layer 0 ≈2×, layer 9 up to ≈6.4×). "More than 4× at 2,048" checks.

### End-to-end throughput (Table 2, gov_report, RTX 4090)

| Pipeline | tok/s | KV reads skipped |
|---|---|---|
| No speculative decoding (FlashAttention) | 42.00 | — |
| Speculative decoding, full attention | **68.26** | — |
| **SpecAttn, p = 0.97** | **59.95** | 71.89% |

> [!warning] The method is net-negative end to end
> SpecAttn is **12% slower than plain speculative decoding** (59.95 vs 68.26 tok/s) at the only length it was tested. The paper attributes this to "high mask generation time (Algo 2)", but Fig. 3 puts the Triton selector at 0.03 ms per call — ~1 ms per round across 30 sparse layers, far too small to explain a ~2 ms/token gap. The uncredited costs are more likely the **CSR conversion** (§ 3.5 concedes it "introduces additional latency"), the per-layer kernel launches, and the fact that at ≈2K context an 8B-class decode step is weight-bound, so a 72% cut in KV reads buys almost nothing — exactly the [[arithmetic-intensity]] argument. The author's own remedy is to test at 10K+ tokens, which was never done.

## Claims that did not check out, or need qualification

- **"Eliminating redundant computation … delivering end-to-end latency improvements"** (abstract, § 2) — **not shown**; Table 2 shows a slowdown. The abstract's headline is a KV-*access* reduction, not a speedup.
- **"Significantly outperforming existing sparse attention methods"** — true for perplexity at matched sparsity vs Quest and StreamingLLM; no speed comparison against Quest is made.
- **Mask-generation time as the cause of the slowdown** — inconsistent with Fig. 3 (see callout).
- **Figure references** — § 4.2 cites "Fig 4" for the sorting-kernel comparison that is captioned Figure 3.
- **Tolerance vs iterations** — Algorithm 2 shows ε = 10⁻⁶ convergence; footnote 1 says the implementation uses 10 fixed iterations.
- Verified: single author / CMU, workshop venue, TinyLlama-1.1B → Llama-2-7B, RTX 4090, PG-19 at 2,048 with 10/90 split, first 2 layers dense, chunk 16, all six Table-1 rows and both relative increases, 42.00 / 68.26 / 59.95 tok/s, 71.89%, ≥4× sorting speedup, >4× kernel speedup at 2,048.

## Limitations

- **≤2,048-token contexts only**, on a 2023 base model, with ~205 prefilled tokens — the regime where decode attention is least important. The author lists 10K+ evaluation as future work.
- **Net slower than the speculative-decoding baseline** it builds on.
- Perplexity is the only quality metric; no downstream tasks, no retrieval probe.
- No production integration (vLLM/PagedAttention is future work); dense CSR mask conversion every round.
- Per-layer (head-aggregated) masks; the draft's own attention is dense over the full context every step.
- v1, workshop track, single author, no code.

## How it differs from STS and from speculative prefill

| | **SpecAttn** (Oct 2025) | [[sts-speculative-token-sparsity-yu-2026\|STS]] (May 2026) | [[speculative-prefill]] (SpecPrefill, [[cross-family-speculative-prefill-upasani-2026\|cross-family]], [[omlx-specprefill-github-2026\|oMLX]]) |
|---|---|---|---|
| Phase accelerated | **decode** (verification pass) | **decode** (verification pass); prefill mode claimed, accuracy-only | **prefill** (TTFT) |
| What the draft's attention selects | KV entries the target reads at *each generation step* | KV entries the target reads at each step, per head | **prompt tokens** the target prefills at all |
| KV of unselected tokens | kept (reads skipped) | kept (reads skipped) | **never created** — dropped for the whole generation |
| FFN cost reduced? | no | no | **yes** (fewer rows) |
| Draft ↔ target alignment | **layer → layer**, KL divergence, monotonic DTW | **head → head**, top-k index overlap | none needed (draft scores tokens; no internal mapping) |
| Selection rule | top-*p* nucleus, sorting-free, union over γ steps | top-*k* budget per head | top-K% chunks by pooled lookahead attention |
| Selection frequency | every speculative round | every speculative round | once, before target prefill |
| Scale tested | Llama-2-7B, ≤2K tokens, RTX 4090; **net slower** | Llama-3.1-8B/70B, to 100K, H20, vLLM; 1.36–2.88× | Llama-3.1-70B/405B, 32K–250K; up to 7.66× TTFT |

The wiki-level point: SpecAttn and STS are the same idea one product cycle apart — draft attention reused as a target sparsity oracle inside the speculative loop — and both are **decode-side [[sparse-attention]]**. They are *not* [[speculative-prefill]], which uses the draft to shrink the prompt before the target ever sees it, and they do not belong in the [[sparse-prefill]] taxonomy any more than [[language-models-can-control-their-own-attention-ho-2026|Declarative Attention]] did.

## Related

- [[sparse-attention]] — where this files (decode-time, lossless KV, draft-model selector).
- [[speculative-decoding]] — the host loop; the draft's attention rows are a by-product it already pays for.
- [[sts-speculative-token-sparsity-yu-2026]] — the same design done at scale, with head mapping, a vLLM integration, and real speedups; does not cite this paper.
- [[sale-low-bit-estimation-sparse-attention-prefill-ji-2025]] — the *prefill*-side "cheap estimator" ingested alongside: INT4 QK products instead of a draft model. Together the three show the estimator can be a low-bit copy of the same model (prefill) or a smaller sibling model (decode).
- [[speculative-prefill]] · [[speculative-prefill-liu-2025]] · [[cross-family-speculative-prefill-upasani-2026]] · [[omlx-specprefill-github-2026]] — the prefill-side use of a draft; contrasted in the table above.
- [[sparse-prefill]] · [[token-pruning]] — where this does **not** go; KV is never dropped here.
- [[kv-cache]] · [[arithmetic-intensity]] · [[llm-inference-roofline-yuan-2024]] — why a 72% cut in KV reads yields no speedup at 2K tokens on a 7B model.
- [[minference-dynamic-sparse-attention-jiang-2024]] — cited as the prefill-side dynamic method whose "benefits diminish … during the token-by-token generation phase".
- [[attention-sink]] — the reason layers 1–2 are kept dense, via Quest.
- [[flash-attention]] — the non-speculative baseline kernel in Table 2.
- [[language-models-can-control-their-own-attention-ho-2026]] — the other decode-side selector the wiki nearly mis-filed.
- [[cmu]] · [[llama]] · [[nvidia]] · [[longbench]]

## References

- [arXiv:2510.27641](https://arxiv.org/abs/2510.27641)
- Quest: Tang et al., ICML 2024, [arXiv:2406.10774](https://arxiv.org/abs/2406.10774) — query-aware KV page selection; the matched-sparsity baseline and the source of the dense-first-2-layers rule.
- Twilight: Lin et al., 2025, [arXiv:2502.02770](https://arxiv.org/abs/2502.02770) — adaptive top-*p* attention sparsity; the selection rule's inspiration.
- StreamingLLM: Xiao et al., [arXiv:2309.17453](https://arxiv.org/abs/2309.17453).
- FlashInfer: Ye et al., [arXiv:2501.01005](https://arxiv.org/abs/2501.01005) — `BlockSparseAttention`.
- Speculative decoding: Leviathan et al., [arXiv:2211.17192](https://arxiv.org/abs/2211.17192); Chen et al., [arXiv:2302.01318](https://arxiv.org/abs/2302.01318).
- MInference: Jiang et al., [arXiv:2407.02490](https://arxiv.org/abs/2407.02490); SpargeAttn: Zhang et al., [arXiv:2502.18137](https://arxiv.org/abs/2502.18137).
- PG-19: Rae et al., [arXiv:1911.05507](https://arxiv.org/abs/1911.05507); LongBench: Bai et al., [arXiv:2308.14508](https://arxiv.org/abs/2308.14508).
- TinyLlama: Zhang et al., [arXiv:2401.02385](https://arxiv.org/abs/2401.02385); Llama 2: Touvron et al., [arXiv:2307.09288](https://arxiv.org/abs/2307.09288).
