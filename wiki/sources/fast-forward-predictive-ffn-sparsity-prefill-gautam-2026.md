---
title: "Fast Forward: Accelerating LLM Prefill with Predictive FFN Sparsity"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2602.00397"
authors:
  - Aayush Gautam
  - Mukul Gagrani
  - Junyoung Park
  - Mingu Lee
  - Chris Lott
  - Narasimha Reddy
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Fast Forward: Accelerating LLM Prefill with Predictive FFN Sparsity

**Aayush Gautam, Mukul Gagrani, Junyoung Park, Mingu Lee, Chris Lott, Narasimha Reddy** (Gautam and Reddy: Texas A&M ECE; Gagrani, Park, Lee, Lott: Qualcomm Technologies; the work was done during Gautam's Qualcomm internship; correspondence Gautam) — [arXiv:2602.00397](https://arxiv.org/abs/2602.00397), cs.LG / cs.AI, submitted 30 Jan 2026, v1 only, 10 pages / 7 figures, CC BY 4.0, ICML submission template. No venue stated, no code. (arXiv's author metadata spells the fifth author "Chiris Lott"; the paper body says Chris.)

> [!important] What this paper is, in one paragraph
> A **trained** system for skipping half of every FFN's neurons during prefill on **SwiGLU models** (Llama-3.2-1B/3B, Llama-3.1-8B, Qwen3-4B) — models that have no ReLU zeros, so "sparsity" here means *the 50% of intermediate neurons with the largest gated-activation magnitude for this 128-token block*, chosen by a small learned predictor before the FFN runs, with the resulting error partly patched by a small learned low-rank corrector, and with a per-layer sparsity budget set from how much attention each layer pays to non-sink tokens. It is the prefill counterpart of the decode-side [[activation-sparsity]] line (Deja Vu, CATS, GRIFFIN) and explicitly *not* a storage trick: all weights stay resident, the gain is fewer FLOPs, and the point of the 128-token block is that one neuron set shared by 128 tokens turns a gather into a dense GEMM. Measured: **1.44× TTFT at 2–4K tokens on an 8B (A100)**, fading to 1.20× at 32K and 1.37× at 1K, for **−6.0% relative on LongBench** (−3.1% at 30% sparsity), with multi-document QA taking most of the hit.

## Problem

The paper's framing is that prefill is where interactive latency lives and the FFN is where prefill's FLOPs live — at the lengths that matter. Attention costs O(T²·d); the FFN costs O(T·d·d_ffn) with d_ffn = 14,336 for Llama-3.1-8B, so the FFN dominates until T ≈ 2·d_ffn ≈ **28K tokens** (their § 2.3 bound, *c* ≈ 2). Production prompt/response ratios they cite make the case: 10:1 for RAG and code completion (SwiftKV, Qiao et al. 2025), and from Preble (Srivatsa et al. 2024):

| Workload | Prompt tokens | Output tokens | Ratio |
|---|---|---|---|
| Programming | 3,871 ± 1,656 | 190 ± 343 | 20.4 : 1 |
| Tool use | 1,835 ± 742 | 43 ± 16 | 42.7 : 1 |
| Embodied agent | 2,285 ± 471 | 16 ± 13 | 142.8 : 1 |

Their own profile of Llama-3.1-8B on an A100 (Fig. 2; per-component time, LayerNorm + QKV + attention + FFN + residual) puts the **FFN at 41% of time at 512 tokens, 53% at 1K, 57% at 2K, 59% at 4K, 56% at 8K, 51% at 16K** (totals 67 / 100 / 174 / 338 / 694 / 1,511 ms). So even a free FFN would cap the end-to-end gain at ~2.4× at 4K and ~2.0× at 16K; halving the FFN caps it around 1.4×. This is the same [[arithmetic-intensity]] picture as [[llm-inference-roofline-yuan-2024]] — prefill is compute-bound in the FFN and projections — read as a target rather than a constraint.

Why the decode-side FFN-sparsity methods do not transfer, in the paper's words: **CATS** "applies token-level weight masking, which breaks the inherent parallelism advantage of prefill" (a different neuron set per token means no shared GEMM); **GRIFFIN** "depends on prompt-level statistics to estimate neuron importance — information unavailable prior to prompt processing" (its "flocking" experts are computed *from* a dense pass over the prompt). And uniform sparsity across layers on long prompts accumulates error. Each of the three components answers one of these.

## Mechanism

Prompts are processed in **128-token blocks** (the block-wise NPU convention they cite from Xu et al. 2025; also the activation-memory argument — a 16K prompt on Llama-3.2-3B needs >1 GB of activations if done at once). Everything below is per block *i*, per layer *l*, on the block input X^{l,i} ∈ ℝ^{128×d}.

### 1. Expert neuron predictor — which neurons, before the FFN runs

A per-layer module with two parts:

- **Attention pooling** with one trainable query vector q_pred ∈ ℝ^{1×d}: the block's token embeddings are keys and values, so a = softmax(q_pred Xᵀ/√d) X is a single d-dimensional summary of the block.
- **Two-layer MLP** s = ReLU(a W₁) W₂ with W₁ ∈ ℝ^{d×r}, W₂ ∈ ℝ^{r×d_ffn}, r = d/16 rounded up to a power of two (**128 for 1B, 256 for 3B and 8B**).

s scores every one of the d_ffn neurons; the **top-K** become the block's mask M. The FFN then runs on gathered sub-matrices — rows of W_up and W_gate, columns of W_down — as a dense computation:

  FFN̂(X) = Ŵ_down · (σ(Ŵ_gate X) ⊙ (Ŵ_up X)),  Ŵ ∈ ℝ^{K×d}.

**How "sparsity" is defined on SwiGLU.** There is no activation that is zero. The predictor's training labels come "from neuron activation magnitudes following the approach in GRIFFIN": in the dense FFN, the neurons whose (gated, post-σ⊙) intermediate activations have the **highest norms over the block** are labelled 1 — the **top 50%** — and the rest 0. So sparsity is *magnitude-based top-K on the gated intermediate, aggregated per 128-token block*, and "50% sparsity" means K = d_ffn/2 neurons kept per block per layer. That is lossy by construction: the other half of the neurons are small, not zero, and their contribution is simply dropped. It is exactly the second row of [[activation-sparsity]]'s table (magnitude thresholding on SiLU), but with the threshold chosen per block rather than per token and *predicted* rather than measured.

**Training.** Weighted binary cross-entropy against those labels. Positives are weighted by rank — the top 20% of positives get weight 32, the next 20% weight 16, "and so on" (so 32/16/8/4/2 across the five quintiles of the kept half); negatives weight 1. The predictor therefore learns to be most right about the largest neurons. The paper states **no dataset, step count, or compute for predictor training** — Minipile is the only corpus named in the paper (for the compensator and the calibration set), so it is the reasonable assumption, but it is an assumption. Table 7's caption describes the predictor as "trained to emulate the Per-Block Dynamic's selections".

*Derived size, Llama-3.1-8B (d = 4,096, d_ffn = 14,336, 32 layers):* q_pred + W₁ + W₂ = 4,096 + 1.05M + 3.67M ≈ **4.72M params/layer, 151M total (~1.9% of the model)**. Its FLOPs are per *block*, not per token — ~9.4 MFLOP per block per layer, i.e. ~0.07 MFLOP/token against a 352 MFLOP/token dense FFN: negligible.

### 2. Error compensation network — patching what the dropped half would have done

A per-layer two-layer FFN with hidden width r′ = d/8 (**512 for 8B**), run **per token** in parallel with the sparse FFN:

  Y = FFN̂(X) + W_c2 · σ(W_c1 · X).

Trained by **layer-wise distillation**: MSE between the dense FFN output Y_dense and the compensated sparse output Y, per layer, base weights frozen ([[knowledge-distillation]] at the sub-layer level). Two phases — warm-started with **oracle masks** (true top-K by activation magnitude), then switched to the trained predictor's masks so it learns the predictor's actual error pattern. **Minipile, 10,000 steps, batch 512.** Input is only X — the compensator does **not** see which neurons were dropped, which the authors name as its main weakness: its corrections have "very small norms", it "applies corrections only along the most confident direction", and it cannot fix a bad expert selection. Ablation: removing it costs 0.58 / **3.20** / 0.41 LongBench points on 1B / 3B / 8B.

*Derived, 8B:* 2·d·r′ = 4.19M params/layer, 134M total; ~8.4 MFLOP/token/layer = 2.4% of the dense FFN, 4.8% of the 50%-sparse one. Predictor + compensator ≈ **285M added parameters, ~3.6% of the 8B model**, and an ideal FFN speedup of 352/(176 + 8.4 + 0.1) ≈ **1.91×** at 50% — the measured 1.82× (below) is 95% of that.

### 3. Layer-wise sparsity scheduler — keyed on "token-mixing importance"

Motivation: the first block carries the [[attention-sink]] tokens, and beyond them, layers differ in how much attention mass they spread over ordinary tokens — a layer that attends broadly is doing more *token mixing* and should be kept denser. The statistic, from a **calibration set of 128 Minipile samples longer than 12K tokens**:

  s_i = (1/|D|·H) · Σ_X Σ_h Σ_{k ∈ non-sink blocks} Σ_t A_i,tk^(h)(X)

— the total attention weight that keys *outside the first block* receive at layer *i*, averaged over heads and samples. **Algorithm 1** turns {s_i} and a global budget B (the keep fraction) into per-layer keep fractions greedily: T ← B·L; for each layer in order, b_i ← min(1, s_i / S_remaining · T), then subtract b_i from T and s_i from S_remaining. Layers with more non-sink attention get a larger share of the compute budget (lower sparsity). Ablation vs uniform 50%: **+1.03 / +2.93 / +0.78 / +0.90%** on Llama 1B / 3B / 8B / Qwen3-4B — real but small ("1–3%" in the text; 0.8–2.9% by the table).

Two more fixed rules: **the first block is always dense** (sinks) and **the last block is always dense** ("helps in QA tasks" — the question usually sits there). These matter far more than the schedule (below).

## Results

Benchmark: **[[longbench]]** English tasks, six categories; models Llama-3.2-1B-Instruct, Llama-3.2-3B-Instruct, Llama-3.1-8B-Instruct, Qwen3-4B. All sparse rows use predictor + compensator + dense first/last block + the layer-wise schedule. Sparsity is the FFN neuron-drop fraction; attention is always dense.

### LongBench (Table 2)

| Model | Sparsity | Single-doc QA | Multi-doc QA | Summ. | Few-shot | Synthetic | Code | **Avg** | **Rel. gap** |
|---|---|---|---|---|---|---|---|---|---|
| Llama-3.2-1B | 0% | 26.76 | 24.74 | 25.43 | 61.35 | 3.59 | 38.18 | 29.88 | — |
| | 30% | 24.80 | 25.07 | 24.77 | 58.97 | 3.31 | 42.01 | 29.58 | −0.99% |
| | 40% | 25.19 | 24.96 | 24.73 | 57.94 | 2.76 | 42.24 | 29.38 | −1.66% |
| | **50%** | 24.63 | 23.04 | 24.39 | 57.38 | 1.48 | 40.71 | **28.37** | **−5.07%** |
| Llama-3.2-3B | 0% | 38.65 | 36.72 | 27.67 | 67.85 | 47.29 | 51.59 | 44.72 | — |
| | 30% | 38.25 | 38.63 | 26.85 | 64.93 | 42.04 | 53.57 | 43.74 | −2.18% |
| | 40% | 37.17 | 35.95 | 27.45 | 65.44 | 40.34 | 53.82 | 43.03 | −3.76% |
| | **50%** | 36.39 | 36.37 | 26.53 | 65.31 | **36.67** | 56.37 | **42.54** | **−4.86%** |
| Llama-3.1-8B | 0% | 43.25 | 45.43 | 29.19 | 69.44 | 54.13 | 59.07 | 49.76 | — |
| | 30% | 42.97 | 40.80 | 28.73 | 67.36 | 52.59 | 59.10 | 48.22 | −3.09% |
| | 40% | 43.86 | 39.09 | 28.47 | 67.09 | 51.64 | 56.08 | 47.40 | −4.75% |
| | **50%** | 42.32 | **38.16** | 28.21 | 66.31 | 51.39 | 56.29 | **46.78** | **−5.99%** |
| Qwen3-4B | 0% | 29.21 | 17.24 | 25.39 | 69.33 | 50.96 | 59.16 | 41.26 | — |
| | 30% | 28.45 | 16.48 | 25.24 | 69.30 | 50.75 | 59.10 | 40.92 | −0.82% |
| | 40% | 28.32 | 15.71 | 25.23 | 68.99 | 50.81 | 59.00 | 40.71 | −1.33% |
| | **50%** | 27.45 | 15.64 | 25.03 | 68.74 | 50.79 | 58.26 | **40.36** | **−2.18%** |

**"<6% loss on LongBench" checks**: the worst cell is −5.99% (8B, 50%); every relative gap recomputes from the averages to within 0.02. Where the loss lands is task-shaped, exactly as [[sparse-prefill]] documents for token-dropping:

- **Llama-3.1-8B: multi-document QA is the casualty** — 45.43 → 38.16 (**−16% relative**) at 50%, and already −4.6 points at 30%. Single-doc QA (−0.9) and summarization (−1.0) barely move. Aggregating across documents is what a half-width FFN loses first.
- **Llama-3.2-3B: synthetic tasks** (passage retrieval/counting) — 47.29 → 36.67 (**−22% relative**).
- **Code improves under sparsity on 1B and 3B** (38.18 → 42.01–42.24; 51.59 → 56.37) — unexplained, and the kind of noise-or-regularization effect that [[pop-prefill-only-pruning-he-2026]] also reports at mild pruning.
- **Qwen3-4B is the most tolerant**: nothing moves more than 1.8 points at 50%. Its dense multi-doc QA (17.24) is already low, so the low relative gap is partly a low baseline.

Sparsity applied to **both prefill and generation** with the same predictor and compensator (Table 3) — the predictor was trained only on 128-token blocks but pools by attention, so it accepts any length: LongBench 28.31 / 42.46 / 46.92 (1B / 3B / 8B at 50%), **MMLU 45.35 / 60.16 / 67.17 vs dense 46.06 / 60.68 / 67.84** — well under a point on [[mmlu]]. (The 8B LongBench score with *more* sparsity, 46.92, is slightly *above* the prefill-only 46.78 — noise.)

### Speed

**Absolute TTFT, Llama-3.1-8B-Instruct, one NVIDIA A100, 50% sparsity (Fig. 1; bars read off the chart, ratios are the figure's labels):**

| Context | Dense TTFT | FastForward | Speedup |
|---|---|---|---|
| 1,024 | ≈0.11 s | ≈0.08 s | **1.37×** |
| 2,048 | ≈0.20 s | ≈0.14 s | **1.44×** |
| 4,096 | ≈0.39 s | ≈0.27 s | **1.44×** |
| 8,192 | ≈0.83 s | ≈0.60 s | **1.39×** |
| 16,384 | ≈1.85 s | ≈1.42 s | **1.30×** |
| 32,768 | ≈4.45 s | ≈3.71 s | **1.20×** |

**End-to-end prefill speedup by sparsity (Fig. 7, read off the curves; the 8B/50% curve coincides with Fig. 1's A100 labels, hardware for the 1B/3B curves is not stated):**

| Model / sparsity | 512 | 1K | 2K | 4K | 8K | 16K | 32K |
|---|---|---|---|---|---|---|---|
| 8B / 50% | ≈1.23 | 1.37 | **1.44** | **1.44** | 1.39 | 1.30 | 1.20 |
| 8B / 40% | ≈1.17 | ≈1.27 | ≈1.32 | ≈1.32 | ≈1.28 | ≈1.22 | ≈1.15 |
| 8B / 30% | ≈1.12 | ≈1.18 | ≈1.21 | ≈1.21 | ≈1.19 | ≈1.15 | ≈1.11 |
| 3B / 50% | ≈1.17 | ≈1.27 | ≈1.31 | ≈1.29 | ≈1.24 | ≈1.17 | ≈1.11 |
| 1B / 50% | ≈1.15 | ≈1.24 | ≈1.27 | ≈1.25 | ≈1.21 | ≈1.15 | ≈1.09 |

The shape is the paper's own explanation and it is consistent: **short contexts lose because the dense first and last blocks are a large fraction of a few blocks** (at 512 tokens, 2 of 4 blocks are dense); **long contexts lose because attention's quadratic term takes over** (the FFN is 51% of time at 16K and falling). The sweet spot is 2K–8K — which is also where the workload table says most prompts are. Smaller models gain less because their FFNs are a smaller share of a (relatively) heavier attention/overhead budget.

**FFN-module speedup, custom CUDA kernels, NVIDIA A5000, 50% sparsity (Fig. 6; the unit of work — per layer, per block — is not stated):**

| Model | Dense FFN | FastForward | Speedup |
|---|---|---|---|
| 8B | ≈20.8 ms | ≈11.4 ms | **1.82×** |
| 3B | ≈10.4 ms | ≈5.8 ms | **1.80×** |
| 1B | ≈5.9 ms | ≈3.4 ms | **1.73×** |

Against the derived ideal of 1.91× (half the FFN plus the compensator), 1.82× says the predictor, gather, and compensator overheads are real but small. The "up to 1.45×" in the abstract is Fig. 7's 8B peak; Fig. 1 labels the same points 1.44×.

### The ablation ladder (Tables 4–7; LongBench average)

Reading the four ablations together gives the contribution of each piece at 50% sparsity:

| Configuration | 1B | 3B | 8B |
|---|---|---|---|
| Uniform 50%, every block sparse | 20.41 | 37.72 | 42.37 |
| + dense first block | 23.99 | 37.40 | 43.43 |
| + dense last block too | 28.08 | 41.33 | 46.42 |
| — the above without the compensator | 27.50 | 38.13 | 46.01 |
| + layer-wise schedule (= Table 2 row) | **28.37** | **42.54** | **46.78** |
| Dense | 29.88 | 44.72 | 49.76 |

So: the **dense last block is the single biggest lever** (+4.1 / +3.9 / +3.0 points) — for QA-style tasks the question lives there. The dense first block helps 1B and 8B and slightly *hurts* 3B (37.72 → 37.40). The compensator is worth 0.4–3.2 points; the schedule 0.3–1.2.

**Predictor quality (Table 7; dense first block only, uniform 50%):**

| Predictor | 1B | 3B | 8B |
|---|---|---|---|
| First-block static (GRIFFIN: pick experts from block 1, reuse for all blocks) | 18.13 | 25.06 | 30.07 |
| **Trained predictor** | **23.99** | **37.40** | **43.43** |
| Per-block oracle (dense FFN, then top-K — infeasible) | 26.02 | 36.45 | 44.75 |

Two conclusions the paper draws that the numbers support. GRIFFIN-style "flocking" — a prompt-wide expert set — **does not hold across prefill blocks**: reusing block 1's neurons costs 13 points on 8B. And the learned predictor is within 1.3–2.0 points of the oracle on 1B/8B and *above* it on 3B ("it learns … from aggregated cross-prompt statistics rather than the local statistics of a single prompt").

## Is it training-free? — plainly, no

Three trained or calibrated artefacts per model, base weights frozen: a **predictor per layer** (weighted BCE on magnitude labels; data and steps unstated), a **compensator per layer** (MSE distillation to the dense FFN, two-phase, Minipile, 10K steps × batch 512), and a **schedule** from a 128-sample calibration pass. Roughly 3–5% of the base model in new parameters (3.6% derived for 8B). Compute cost in GPU-hours is never given. This is the opposite of [[pop-prefill-only-pruning-he-2026]], which trains nothing.

## Relation to the wiki's activation-sparsity and weight-streaming line

[[activation-sparsity]] distinguishes ReLU zeros (exact, [[llm-in-a-flash-alizadeh-2023]]), magnitude thresholding on SiLU/GELU (lossy, [[neuroprefetcher-delta-prefetching-dhar-2026]]) and MoE routing (exact, trained). Fast Forward is a fourth entry, and it is worth being exact about how it differs from the two storage papers:

| | [[llm-in-a-flash-alizadeh-2023]] | [[neuroprefetcher-delta-prefetching-dhar-2026]] | Fast Forward |
|---|---|---|---|
| Phase | decode | decode | **prefill** (decode optional) |
| What sparsity saves | **bytes** from flash (weights do not fit) | bytes from NVMe (weights do not fit) | **FLOPs** (weights resident) |
| Sparsity source | ReLU zeros, ~97%, exact | percentile magnitude threshold on SwiGLU, 31–62%, clustered to 512 centroids | **learned top-K on SwiGLU magnitude, 30–50%, per 128-token block** |
| Granularity | per token, per layer | per token, all layers at once | **per block of 128 tokens**, per layer |
| Predictor | per-layer low-rank, ~1.25% of model | one-shot trunk + 62 heads, 2.86% | per-layer attention-pool + MLP, ~1.9% (derived) |
| Error handling | none needed (exact) | none (accuracy loss 7–32%) | **trained compensator**, +0.4–3.2 pts |
| Quality at operating point | lossless | −10–12% HellaSwag at the 9.5× point | −6% relative LongBench at 1.44× |
| Hardware | Apple M1 Max / RTX 4090 | Jetson AGX Orin | A100, A5000 (NPU is the motivation, never measured) |

The block granularity is the conceptual move. A per-token neuron set (CATS, Deja Vu, both flash papers) is fine at decode — there is one token — but at prefill it turns each FFN into 128 different gathers. Sharing one set across 128 tokens keeps a single dense GEMM at half width, which is what the custom kernels exploit for 1.82×. The price is that the set must suit *all* 128 tokens, which is why K = 50% is as far as they go, why the compensator exists, and why the loss is not zero.

The paper's own last paragraph closes the loop back to [[weight-streaming]]: loading a different expert set per block "is intensive on memory bandwidth", and a stronger compensator might allow "a single, static set of experts … for all subsequent computations", which "would drastically reduce memory bandwidth requirements". On a resident-weights GPU that bandwidth is on-chip and second-order; on the storage-backed systems it *is* the cost — so a static-expert Fast Forward would be a prefill-side match for the flash-streaming decoders, and none exists yet.

## Claims that did not check out, or need qualification

- **"Keeping the first and last blocks dense improves performance by 15–30%"** (§ 6.2): from Table 5, 20.41 → 28.08 is **+37.6%** (1B), 37.72 → 41.33 is **+9.6%** (3B), 42.37 → 46.42 is **+9.6%** (8B). Neither bound holds.
- **Table 5 caption says "lower is better"** — the metric is LongBench average, higher is better. The caption also refers to a "+ w/ Dense Last" row the table does not have (it has "+ w/ Dense First").
- **"Up to 1.45× … corresponding to a 45% reduction in FLOPs"** (§ 5): a 1.45× speedup is a 31% time reduction, and at FFN ≈ 59% of time, 50% FFN sparsity is ~30% of FLOPs. Fig. 1 labels the peak 1.44×.
- **"FFN layers can account for more than two-thirds of the total computation"** (§ 1) vs. the paper's own Fig. 2, where the FFN's share of *time* peaks at 59% and is 41% at 512 tokens. Parameter share and FLOP share at short T can exceed two-thirds; measured time does not.
- **Fig. 2's caption says "a transformer block"** but the totals (1,511 ms at 16K) are whole-model magnitudes, slightly under Fig. 1's TTFT (1.85 s) — consistent with a whole-model per-component sum that excludes embedding/LM head. Caption, not data.
- **"Smaller sparsity levels (30–40%) reduce this gap to below 2–4%"**: 8B at 40% is −4.75%.
- **No external baseline at all** — CATS, TEAL, GRIFFIN, Deja Vu appear only as motivation; GRIFFIN's first-block static is an ablation arm, not a tuned baseline; no comparison to token pruning or sparse attention at the same TTFT.
- **Edge-NPU deployment is the stated motivation (Qualcomm, block-wise NPU processing, phones and laptops), but every number is from an A100 or A5000.** No NPU measurement, no memory-bandwidth or energy result.
- **Predictor training data/steps/cost unstated**; total pipeline cost unstated.
- Everything else verified: all 16 relative gaps; Table 3 cells; the ablation ladder is internally consistent across Tables 4–7 (the same configurations recur with identical numbers); Fig. 1 ratios match the bar heights; r = 128 / 256; r′ = d/8; 128-token blocks; 128 calibration samples > 12K; 10K steps × 512; the 28K crossover follows from T < 2·d_ffn.

## Limitations

- **Lossy by design, and task-shaped.** Multi-doc QA (−16% on 8B) and synthetic retrieval/counting (−22% on 3B) pay for the average; anything that needs aggregation across the whole context is the wrong workload, as with every method under [[sparse-prefill]].
- **Ceiling ~1.45× and a narrow window** (2K–8K). At 32K it is 1.20× and falling; below 1K the dense first/last blocks eat the gain. It does nothing for attention, so at 100K+ [[sparse-attention]] is the lever, not this.
- **Trained per model**, with three artefacts and a custom kernel — not a drop-in serving flag.
- **The compensator is blind to the mask** (the authors' own headline limitation) and its corrections are small; the 3B model is the one it rescues (+3.2), the others barely need it.
- **Dynamic expert loading per block is bandwidth-heavy** on the very edge hardware the paper targets; the static-expert variant that would fix it is future work.
- Four models ≤ 8B, one benchmark family plus MMLU, no MoE, no GELU model, no batch >1 study, no NPU.

## Related

- [[pop-prefill-only-pruning-he-2026]] — the depth-axis twin: skip the last third of layers at prefill only, training-free, 1.36–1.37× at 2K on Llama-3.1-8B / Gemma-3-12B with −0.1 to −1.7 average points. Same premise (prefill tolerates what decode cannot), opposite engineering (static and free vs learned and lossy), untested in combination — and in principle multiplicative, since one removes layers and the other halves the FFN inside the remaining ones.
- [[activation-sparsity]] · [[weight-streaming]] · [[llm-in-a-flash-alizadeh-2023]] · [[neuroprefetcher-delta-prefetching-dhar-2026]] — tabulated above. This source is the first prefill-side entry for the activation-sparsity page and the argument that block-level sharing is what makes neuron sparsity GEMM-friendly.
- [[methods-to-speed-up-prompt-processing]] · [[sparse-prefill]] — the taxonomy there is token-axis and attention-axis; FFN-width sparsity is a model-axis lever that keeps every token and every KV entry, which removes the "dropped tokens never get KV" hazard of [[token-pruning]] and [[speculative-prefill]] at the cost of being lossy on every token instead.
- [[mixture-of-experts]] — the trained, exact version of "only some of the FFN runs"; Fast Forward's "experts" are individual neurons picked post hoc from a dense FFN, and the router is learned after the fact. An MoE model gets its FFN sparsity for free at prefill already.
- [[attention-sink]] — why the first block is dense and why the scheduler excludes it from the token-mixing statistic.
- [[chunked-prefill]] · [[sarathi-chunked-prefill-agrawal-2023]] — the 128-token blocks *are* chunked prefill, cited as such; the sparsity rides on the memory-bounding use of chunking, not the scheduling one.
- [[longbench]] · [[long-context-llm]] · [[mmlu]] — evaluation.
- [[arithmetic-intensity]] · [[llm-inference-roofline-yuan-2024]] · [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]] — the compute-bound-prefill premise, and the reason the FFN share sets the ceiling.
- [[knowledge-distillation]] — the compensator is sub-layer distillation to the dense FFN.
- [[edge-inference]] · [[hardware-accelerator]] · [[nvidia]] · [[qualcomm]] (no page) — the target is Qualcomm NPUs; the evidence is A100/A5000.
- [[llama]] · [[qwen]] — models. [[layer-skipping]] — the paper's own "layer skipping" citations (FFN-SkipLLM, He 2025, Luo 2025) are the decode-side relatives of POP.
- [[prompt-compression]] · [[llmlingua-jiang-2023]] · [[flash-attention]] — the "complementary directions" the intro sets aside.

## References

- [arXiv:2602.00397](https://arxiv.org/abs/2602.00397) (v1, 30 Jan 2026).
- GRIFFIN: Dong et al. 2024 — prompt-level "flocking" experts; the label recipe and the static-expert ablation arm. CATS: Lee et al. 2024 — per-token thresholded SwiGLU sparsity, the "breaks parallelism" contrast. Deja Vu: Liu et al., ICML 2023 — contextual sparsity. ShadowLLM: Akhauri et al. 2024; TEAL and other decode-side sparsity cited alongside.
- Sarathi: Agrawal et al., [arXiv:2308.16369](https://arxiv.org/abs/2308.16369); Xu et al. 2025 (fast on-device LLM inference with NPUs, ASPLOS) — the 128-token block convention.
- StreamingLLM: Xiao et al. 2024; Barbero et al. 2025 ("Why do LLMs attend to the first token?") — attention sinks.
- Preble: Srivatsa et al. 2024; SwiftKV: Qiao et al. 2025 — workload prompt/decode ratios.
- FFN-SkipLLM: Jaiswal et al. 2024; SliceGPT: Ashkboos et al. 2024; SparseGPT: Frantar & Alistarh 2023 — the layer-skipping and static-pruning lines set aside.
- LongBench: Bai et al., [arXiv:2308.14508](https://arxiv.org/abs/2308.14508). Llama 3: Grattafiori et al. 2024; Qwen3: Yang et al. 2025. Minipile (training/calibration corpus).
