---
title: "SALE: Low-bit Estimation for Efficient Sparse Attention in Long-context LLM Prefilling"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2505.24179"
authors:
  - Xiaodong Ji
  - Hailin Zhang
  - Fangcheng Fu
  - Bin Cui
year: 2025
status: complete
importance: medium
tags:
  - 2025
  - paper
---

# SALE: Low-bit Estimation for Efficient Sparse Attention in Long-context LLM Prefilling

**Xiaodong Ji, Hailin Zhang, Fangcheng Fu, Bin Cui** ([[peking-university|Peking University]]) — [arXiv:2505.24179](https://arxiv.org/abs/2505.24179), cs.LG / cs.AI, submitted 30 May 2025, v1 only, 16 pages, CC BY 4.0. No venue stated on arXiv. **Code released:** [github.com/BirdChristopher/SALE](https://github.com/BirdChristopher/SALE) (stated in the abstract).

> [!important] What this method is, in one paragraph
> A **training-free, prefill-only block-[[sparse-attention]]** kernel. Instead of guessing the sparsity pattern from a coarse proxy (a few probe queries as in [[minference-dynamic-sparse-attention-jiang-2024|MInference]], or pooled "representative tokens" per chunk as in FlexPrefill / SpargeAttn), SALE **estimates every entry of the attention map** using **INT4-quantized Q and K** on low-bit Tensor Cores, scores each entry with a *Relative Attention Score* computed against the row's sink-local region, and skips any 64×32 block whose maximum score falls below a per-head calibrated threshold τ. The estimator costs **~11% of FlashAttention-2's attention time at 64K–128K** (but **24% at 8K**); the resulting sparse attention pass runs **≥3.36× faster than FA2 on Llama-3.1-8B past 64K**. Everything is measured as **attention time only** on an RTX 4090 — this is a kernel speedup, not a TTFT speedup, and it lives squarely in the [[sparse-prefill]] family that keeps every token and every KV entry.

## Problem

Prefill attention is O(N²) and, per the authors, "the primary bottleneck" for long inputs. Existing dynamic sparse attention gets its pattern from a **coarse inspection** of the map:

- **Line-pattern methods** (MInference, SampleAttention) decompose the pattern into vertical and slash lines and place them by looking at the exact attention scores of a *subset* of query tokens (MInference's last-64-queries probe).
- **Block methods** (FlexPrefill, SpargeAttn, HiP Attention) form a representative token per chunk and decide which blocks to keep from the products of representatives.

Both "fail to achieve a satisfactory accuracy-efficiency trade-off since they cannot perform fine-grained inspection of the entire attention map". The paper's answer is to inspect all N² entries, and to make that affordable by doing it at 4 bits. Its stated non-goals: retraining-based routes (native sparse attention, [[linear-attention]], [[state-space-model|SSMs]]) and the *decode*-side approximators (SparQ, InfiniGen, retrieval over the [[kv-cache]]); and it calls [[token-pruning]]-style context dropping "orthogonal" and notes it "severely degrade[s] generation quality when processing complex inputs".

## Mechanism

Three passes per layer: **Quantization → Selection-Pass → Computation-Pass**.

**1. 4-bit attention-weight approximation.** Q and K are quantized to INT4 with the per-thread scheme from SageAttention-2 (Triton kernel; "quantization overhead is negligible" — Table 3 says 11–208 ms across 8K–128K, i.e. 5–10% of FA2 time). The Selection-Pass then walks the map exactly like FlashAttention-2's loop nest — outer loop over query blocks (b_q = 64), inner over key blocks (b_k = 32) — but computes Q̃K̃ᵀ on INT4 Tensor Cores instead of FP16, reading 4× fewer bytes.

**2. Relative Attention Score.** Softmax scores need the full row sum, which a sparse kernel never has. SALE instead uses the observation (from [[attention-sink]] / StreamingLLM-style work) that the **sink-local region** of every row — the first 32 tokens and the last 128–256 — reliably carries high attention. For each query block it first computes *full-precision* attention on those sink-local key blocks (which are always kept), obtaining a running max m̃ᵢ and sum l̃ᵢ over that region, and then scores each middle entry as

  P̃[i,j] = exp(S̃[i,j] − m̃ᵢ) / l̃ᵢ

i.e. "how big is this entry relative to the attention mass the row already puts on its sink and local window". A block is kept iff max P̃ ≥ τ (τ ≈ 0.004 typical). Because this is a comparison against a *threshold*, not a top-k, the sparsity level **adapts per input** — sparser maps skip more.

**3. Kernel tricks.** (a) With per-thread quantization scales, the largest relative score and the largest raw INT4 product coincide, so **only one value per thread is dequantized**. (b) The comparison exp(S̃ − m̃)/l̃ ≥ τ is rewritten as **S̃ ≥ ln(τ·l̃) + m̃**, one float compare and no exp/div in the inner loop. (c) Thread-block all-reduce of the keep/skip decision is amortised over segments of four key blocks (App. C) — which is why the local window is "≥128, ≤256" rather than exact. The Computation-Pass is the SpargeAttn block-sparse kernel with SageAttention's INT8 QKV quantization.

**4. Per-head threshold calibration (offline).** Heads differ in sparsity, so a shared τ is suboptimal. For each head, start at τ₀ = 0.008 and **halve τ** until the L1 error ‖O − Õ‖₁/N against full attention drops below an error bound θ (θ = 0.4 for Llama-3.1, 2.0 for Qwen-2.5). Calibration uses **five Retrieve.KV samples from InfiniteBench** and takes **~5 minutes** for Llama-3.1 on the 4090 server. Fig. 3(b) shows the calibrated version dominating the uncalibrated one (≈+20 InfiniteBench points at equal 128K latency). θ is the single knob that trades sparsity for accuracy.

Nothing is trained; integration is "trivial code modifications" — the HF `transformers` attention module is swapped.

## Setup

| | |
|---|---|
| Models | **Llama-3.1-8B-Instruct** (most experiments), **Qwen2.5-32B-Instruct**; both 128K context |
| Hardware | server with **8× GeForce RTX 4090** (Ada), *no* tensor- or context-parallelism; INT4 Tensor Cores required |
| Baselines | FlashAttention-2 (dense), MInference (γ from its released config; all heads Vertical-Slash), FlexPrefill (γ = 0.95, BF16), SpargeAttn (l₁ = 0.08, l₂ = 0.09 Llama; 0.04/0.05 Qwen) |
| Block size | b_q = 64, b_k = 32; sink 32 tokens; local 128–256 tokens |
| Latency metric | **attention time across all 32 layers during prefill only**, including quantization, selection and index ops; "E2E" = that, summed over a dataset |
| Decoding | greedy |

## Results

### Attention speedup vs. context length (Fig. 3a, Table 3; Llama-3.1-8B, RTX 4090, one NIAH input truncated to each length)

| Context | Quant + Select (ms) | Compute-Pass (ms) | SALE total (ms) | FA2 (ms) | **SALE vs FA2** | Estimator overhead / FA2 | MInference | FlexPrefill | SpargeAttn |
|---|---|---|---|---|---|---|---|---|---|
| 8K | 11 + 14 | 51 | 76 | 106 | **1.39×** | **23.9%** | ≈0.07× | ≈0.42× | **≈1.60×** |
| 16K | 21 + 48 | 137 | 206 | 416 | **2.02×** | 16.7% | ≈0.22× | ≈1.0× | ≈1.97× |
| 32K | 47 + 166 | 378 | 591 | 1,597 | **2.70×** | 13.3% | ≈0.53× | ≈1.48× | ≈2.45× |
| 64K | 99 + 634 | 1,117 | 1,850 | 6,224 | **3.36×** | 11.5% | ≈1.08× | ≈2.21× | ≈3.10× |
| 128K | 208 + 2,562 | 3,599 | 6,369 | 24,731 | **3.88×** | 11.1% | ≈2.25× | ≈2.80× | ≈3.60× |

Bold totals are derived from Table 3 (sum of the three SALE stages ÷ FA2) and agree with the Fig. 3(a) bars to ±0.03; the baseline columns are read off Fig. 3(a). Checks: **"at least 3.36× past 64K"** is exactly the 64K cell and the 128K cell is higher; **"≈11% overhead"** is the 64K–128K value, and the Computation-Pass alone is 5.57× / 6.87× faster than FA2 at those lengths. Two things the abstract does not say: the overhead is **24% at 8K**, and **SpargeAttn is faster than SALE at 8K** (1.60× vs 1.39×) — the paper's own hedge is "outperforming all sparse attention baselines *in most cases*".

### Accuracy (Tables 1–2, Fig. 2; speedup column = 64K NIAH input)

| | Llama-3.1-8B: FA2 | MInfer | Flex | Sparge | **SALE** | Qwen2.5-32B: FA2 | MInfer | Flex | Sparge | **SALE** |
|---|---|---|---|---|---|---|---|---|---|---|
| [[longbench\|LongBench]] avg (16 tasks) | 48.77 | 47.03 | 46.18 | 47.48 | **48.39** | 50.85 | 51.29 | 49.88 | 50.57 | **51.30** |
| [[infinitebench\|InfiniteBench]] avg (5 tasks) | 37.75 | 27.47 | 33.55 | 35.83 | **37.11** | 28.66 | 30.70 | 29.50 | 28.02 | **30.92** |
| InfiniteBench Retrieve.KV | 55.60 | 20.00 | 38.00 | 47.20 | **56.40** | 4.00 | 7.60 | 4.80 | 4.60 | 5.40 |
| [[needle-in-a-haystack\|NIAH]] avg (Llama only) | 96.1% | 94.1% | 94.5% | 95.0% | **96.0%** | — | — | — | — | — |
| Attention speedup at 64K | 1.00× | 1.07× | 2.21× | 3.11× | **3.36×** | 1.00× | 1.25× | 1.39× | 2.55× | **3.28×** |
| NIAH E2E speedup | 1.00× | 1.41× | 3.35× | 3.35× | **3.81×** | — | — | — | — | — |

- **Llama-3.1-8B loses 0.38 LongBench and 0.64 InfiniteBench points**; the per-task damage is concentrated in **2WikiMQA (46.16 → 42.61)**, **Math.Find (34.29 → 30.57)**, En.QA (15.12 → 13.19), TREC (72.5 → 70.5) and TriviaQA (91.65 → 90.47). Retrieval tasks are untouched or improve (LongBench passage retrieval 99.5 → 100.0; Retrieve.KV 55.6 → 56.4; NIAH −0.1 pt) — the sink-local-relative criterion is, by construction, good at keeping the few sharp columns a retrieval head needs.
- **Qwen2.5-32B improves** on both averages (+0.45 / +2.26), which the authors attribute to filtering noise; note MInference also improves on Qwen, so part of this is that Qwen-32B's dense baseline is itself noisy on these long inputs (Retrieve.KV at 4.0).
- MInference is the clear loser here on Llama (InfiniteBench 37.75 → 27.47; Retrieve.KV 55.6 → 20.0) — using its *released* per-head config with all heads Vertical-Slash, on a 4090.
- Fig. 4 trade-off curves (E2E attention latency, summed over the benchmark): on LongBench SALE sits at ~300–700 s at ≥ FA2 score, FlexPrefill at ~700–1,100 s below it, MInference at ~3,400–4,800 s; on InfiniteBench SALE reaches the FA2 line at ~9,800 s where SpargeAttn needs ~18,000 s and MInference ~20,000 s. SALE dominates on both.
- **Ablation, App. E:** estimating with full-precision (16-bit) QK instead of INT4 *increases* sparsity slightly and *lowers* InfiniteBench slightly at the same θ — INT4 error makes some estimates overshoot, so more blocks are kept; the cost is the Selection-Pass overhead, which balloons without INT4 (Fig. 5a).

## Where SALE stops paying — the shorter-context question

Every number above is **attention time only**. The [[sparse-prefill]] rule says attention is ~¼ of an 8B model's prefill at ~10K, ~70–80% at 128K. Applying Amdahl to Table 3 (share assumed ¼ / ⅓ / ½ / ⅔ / ¾ at 8K / 16K / 32K / 64K / 128K — *derived*, not measured by the paper):

| Context | attention speedup (measured) | implied TTFT gain (derived) |
|---|---|---|
| 8K | 1.39× | **~1.07×** |
| 16K | 2.02× | ~1.2× |
| 32K | 2.70× | ~1.45× |
| 64K | 3.36× | ~1.8× |
| 128K | 3.88× | ~2.3× |

So SALE is **never net-negative in this data** — unlike MInference (0.07× at 8K) and FlexPrefill (0.42× at 8K), which are dramatically slower than dense below 16K in the same measurement — but below ~16K it is a single-digit-percent TTFT win, because the 30 ms it saves at 8K is against a ~400 ms prefill and its estimator alone eats 24% of the attention time. The break-even against *doing nothing* is somewhere around 4K (extrapolating the overhead-ratio trend: 24% at 8K, rising as N falls). The practical reading for the wiki's decision rule: SALE moves the "sparse attention is Amdahl-bounded" floor down from ~64K to ~16–32K, and it is the first of the three coarse-vs-fine methods here that does not lose to FA2 at 8K; it does **not** change the fact that only token-dropping shrinks the FFN. It also composes: the paper calls token pruning/merging "orthogonal", and the natural stack is a draft-picked subset ([[speculative-prefill]]) with SALE on the survivors, the same shape as [[draft-based-approximate-inference-galim-2025|SpecKV-PC]].

> [!warning] Hardware dependency the abstract omits
> The 11% overhead depends on **INT4 Tensor Core GEMM**, which Ada (RTX 4090) and Ampere have. App. A concedes the method "may lose its performance advantage on hardware that does not support efficient 4-bit matrix multiplication" and that FP4 (Blackwell) or LUT-based low-bit GEMM would need "additional adaptations". To my knowledge Hopper (H100) dropped INT4 Tensor Core support, so the datacenter GPU most likely to run 128K prefill is the one where the estimator's cost is *unmeasured*. Every latency number here is from a consumer Ada card.

## Claims that did not check out, or need qualification

- **"At least 3.36× for sequences longer than 64K"** — checks (3.36× at 64K, 3.88× at 128K), but it is the *attention-kernel* speedup on Llama-3.1-8B; Qwen2.5-32B gets 3.28× at 64K, and TTFT gains are roughly half (derived above).
- **"≈11% of the full attention latency"** — checks at 64K–128K (11.5%, 11.1%); it is **13.3% at 32K, 16.7% at 16K, 23.9% at 8K**.
- **"Outperforming all sparse attention baselines"** — not at 8K, where SpargeAttn is faster (1.60× vs 1.39×). The text says "in most cases".
- **"Negligible loss in model accuracy"** — averages support it (−0.4 / −0.6 on Llama), but individual tasks lose 3.5–3.7 points (2WikiMQA, Math.Find).
- **Table 3 overhead ratios** — (11+14)/106 = 23.6%, (99+634)/6224 = 11.8% vs the stated 23.9% / 11.5%; rounding of the ms entries, not an error.
- MInference's baseline numbers (0.07× at 8K; Retrieve.KV 20.0) are far worse than in MInference's own paper — different GPU (4090 vs A100), released config, all-Vertical-Slash. Treat the baseline gaps as "as shipped on a 4090", not as MInference's ceiling.
- Everything else verified: author list, PKU, code URL, INT4 via SageAttention-2, b_q/b_k = 64/32, sink 32 / local 128–256, τ₀ = 0.008 halving, θ = 0.4 / 2.0, five Retrieve.KV calibration samples, ~5 min calibration, 3.81× / −0.1 pt NIAH, LongBench and InfiniteBench averages, 64K speedup row for both models, Computation-Pass 2.08–6.87×.

## Limitations

- **Attention-only latency, single consumer GPU, batch 1**, no TTFT or end-to-end prefill numbers, no throughput/serving measurement.
- **Requires INT4 Tensor Cores**; Hopper and FP4-only parts are unaddressed (App. A).
- **Per-model offline calibration** (5 samples, 5 minutes, one θ) — cheap, but "training-free" is not "configuration-free", and the calibration set is a retrieval task, which may explain why retrieval survives best.
- **Prefill only.** Decode attention is left dense; nothing here reduces [[kv-cache]] size or decode bandwidth — every token keeps its KV entry, which is exactly why it composes with, rather than replaces, token dropping.
- Two models, one family each; no ablation of block size, sink size, or the sink-local assumption on models without strong attention sinks.
- v1 only; no peer-review venue stated.

## Related

- [[sparse-attention]] · [[sparse-prefill]] — the family and the umbrella. SALE is a **fine-grained estimator** answer to *who decides the pattern*: not a probe of a few queries (MInference), not pooled chunk representatives (FlexPrefill, SpargeAttn — **no wiki pages**), but a 4-bit look at every entry.
- [[minference-dynamic-sparse-attention-jiang-2024]] — the main baseline; SALE's measurement of it (0.07× at 8K, 1.08× at 64K on a 4090) is the harshest in the wiki and sharpens that page's "index overhead 30% at 10K" into "net loss below 64K on consumer hardware".
- [[token-sparse-attention-jo-2026]] — the other 2026 fine-grained-selection method in the wiki; it gathers whole tokens per head, SALE gathers blocks per (query-block, key-block); both stack on the coarse methods.
- [[flash-attention]] — the dense baseline and the loop structure the Selection-Pass copies.
- [[attention-sink]] — the sink-local region that the Relative Attention Score normalises against.
- [[quantization]] — INT4 used for *estimation* only; outputs are computed at INT8 (SageAttention) precision.
- [[speculative-prefill]] · [[token-pruning]] · [[draft-based-approximate-inference-galim-2025]] — the orthogonal, FFN-shrinking levers; the natural composition partner.
- [[methods-to-speed-up-prompt-processing]] · [[long-context-llm]] · [[arithmetic-intensity]] · [[llm-inference-roofline-yuan-2024]] — why attention share, not kernel speedup, is the number that matters.
- [[claa-cross-layer-attention-aggregation-mcdanel-2026]] — the source of the wiki's "¼ at 10K" figure used in the derived TTFT table.
- [[fast-prefill-fpga-sparse-attention-jayanth-2026]] — sparse prefill attention on FPGA; a different answer to SALE's INT4-hardware dependency.
- [[sts-speculative-token-sparsity-yu-2026]] · [[specattn-speculating-sparse-attention-shah-2025]] — the two *draft-model* estimators ingested alongside this one. Both run inside a [[speculative-decoding]] loop and measure only **decode**-side gains (STS claims a prefill mode but never times it), so they are the decode counterparts, not competitors, of SALE.
- [[llama]] · [[qwen]] · [[nvidia]] · [[peking-university]]
- [[needle-in-a-haystack]] · [[infinitebench]] — `infinitebench` has no page yet.

## References

- [arXiv:2505.24179](https://arxiv.org/abs/2505.24179) · code [github.com/BirdChristopher/SALE](https://github.com/BirdChristopher/SALE)
- MInference: Jiang et al., [arXiv:2407.02490](https://arxiv.org/abs/2407.02490) — [[minference-dynamic-sparse-attention-jiang-2024]].
- FlexPrefill: Lai et al., ICLR 2025, [arXiv:2502.20766](https://arxiv.org/abs/2502.20766) — context-aware per-head block sparsity via representative tokens; γ = 0.95 here.
- SpargeAttn: Zhang et al., [arXiv:2502.18137](https://arxiv.org/abs/2502.18137) — the block-sparse compute kernel SALE builds on.
- SageAttention / SageAttention-2: Zhang et al., [arXiv:2410.02367](https://arxiv.org/abs/2410.02367), [arXiv:2411.10958](https://arxiv.org/abs/2411.10958) — INT8 attention and the per-thread INT4 quantization.
- FlashAttention-2: Dao, [arXiv:2307.08691](https://arxiv.org/abs/2307.08691) — [[flash-attention]].
- StreamingLLM: Xiao et al., [arXiv:2309.17453](https://arxiv.org/abs/2309.17453) — the sink observation behind the Relative Attention Score.
- LongBench: Bai et al., [arXiv:2308.14508](https://arxiv.org/abs/2308.14508); InfiniteBench: Zhang et al., [arXiv:2402.13718](https://arxiv.org/abs/2402.13718); Needle-In-A-Haystack: gkamradt.
