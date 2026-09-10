---
title: "POP: Prefill-Only Pruning for Efficient Large Model Inference"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2602.03295"
authors:
  - Junhui He
  - Zhihui Fu
  - Jun Wang
  - Qingan Li
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# POP: Prefill-Only Pruning for Efficient Large Model Inference

**Junhui He, Zhihui Fu, Jun Wang, Qingan Li** (He: Wuhan University + OPPO Research Institute; Fu, Wang: OPPO Research Institute; Li: Wuhan University, corresponding) — [arXiv:2602.03295](https://arxiv.org/abs/2602.03295), cs.CL / cs.AI / cs.CV, v1 3 Feb 2026, **v2 16 Apr 2026** (this summary), CC BY 4.0. NSFC 62472330. No venue stated (the acknowledgments thank "all the reviewers", so it has been through at least one review round). No code link.

> [!important] What this paper is, in one paragraph
> A **training-free** way to make prefill ~1.36× cheaper on 8B–12B dense models by **not running the last third of the layers during prefill** — and *only* during prefill. The justification is a measured asymmetry: with a "virtual gate" per layer, the loss-sensitivity of deep layers is high when the layer serves next-token prediction (decode) and near zero when it only serves context encoding (prefill). Two fixes make the stage split work: (1) the skipped layers still run their own pretrained K/V projections on the (unchanged) residual stream, so the decode-side attention finds a complete [[kv-cache]]; (2) the **last prompt token is handed to the full model** as the first decode step, because the first generated token needs the deep layers. Result: ≈33% of layers skipped for prefill at **0.04–1.7 avg-point loss** on a 12-benchmark suite, where whole-model layer pruning (ShortGPT, SliceGPT at 25%) collapses to 0–1% on GSM8K. Decode and peak memory are untouched — this buys TTFT only.

## Problem

Structured pruning (removing whole layers or channels) is hardware-friendly but, in the authors' framing, fails in practice because it is **stage-agnostic**: the same reduced architecture serves both prefill and decode. They argue the two stages have different jobs — prefill only has to *encode the prompt into the KV cache*; decode has to *both* encode the current token *and* produce the next-token distribution — and so should have different tolerance to pruning. The evidence that stage-agnostic pruning is broken is stark in their own Table 1: ShortGPT and SliceGPT at 25% keep likelihood-scored benchmarks (MMLU, HellaSwag) in a plausible range but drop **GSM8K from 79.68% to 0.38% / 0.91%** and **HumanEval to 0.00%** on Llama-3.1-8B — the pruned model can rank options but cannot *generate*.

This is the prefill analogue of the split the wiki already tracks under [[methods-to-speed-up-prompt-processing]]: prefill is compute-bound at ~1,000 OPs/byte ([[llm-inference-roofline-yuan-2024]], [[arithmetic-intensity]]), so the only levers are to skip work, shrink work, or make FLOPs cheaper. Every method under [[sparse-prefill]] shrinks the *sequence* (tokens) or the *attention matrix*. POP shrinks the **model depth**, for the prefill pass only, and keeps every token.

## Mechanism

### 1. Virtual gates — how layer importance is measured

Layer *l*'s residual branches are multiplied by a scalar **virtual gate** *g_l* (Molchanov et al. 2019 style):

  ŷ_l = x_l + Attn(x_l) ⊙ g_l,  x̂_{l+1} = ŷ_l + FFN(ŷ_l) ⊙ g_l

*g_l* = 1 is the original model; *g_l* = 0 is an identity skip of the whole layer (attention + FFN, residual kept). Importance is defined as the expected loss increase when *g_l* goes 1 → 0, and estimated by a second-order Taylor expansion around *g_l* = 1. The trick that makes this a single forward-backward pass: targets are **sampled from the model's own distribution** (ŷ ~ P_θ(·|x)) rather than taken from ground truth. Under that sampling the expected first-order term vanishes and the expected Hessian equals the Fisher information (the squared gradient), so

  Ĩ_l = E[(∂L/∂g_l)²].

No layer is ever physically removed during the analysis — *L* separate ablation passes become one backward pass per calibration sample.

### 2. Stage-aware analysis — the asymmetry

The gate is split into two parameters, *g^prefill* and *g^decode*. Prefill is defined as the encoder E over the prompt x_{1:N−1} that emits the KV cache; decode is D, which consumes that cache plus the generated history x_{N:t} to predict x_{t+1}. The loss is cross-entropy over all output tokens t = N … T−1, and one backward pass yields both Ĩ_l^prefill and Ĩ_l^decode.

Measured on **Llama-3.1-8B-Instruct / WizardLM-V2-196K** and **Qwen3-VL-8B-Instruct / LLaVA-Instruct-150K** (Fig. 1 — a plot, no numeric values given), three consistent findings:

| Finding | What the curves show |
|---|---|
| **Stage disparity** | Decode importance exceeds prefill importance for the majority of layers — decode is the fragile stage. |
| **Deep layers critical for decode** | Decode importance rises with depth; the final layers "often exceed the visualization range". |
| **Deep layers redundant for prefill** | Prefill importance is moderate for the first layers, declines through the middle, and is "approaching zero" for the final layers. |

So the paper's asymmetry claim is exactly: *the same deep layer is essential when its output feeds the LM head and dispensable when its output only feeds the KV cache of later tokens.*

### 3. Prefill-only pruning — what is skipped

A **static** rule: skip the **last 1/3 of layers during prefill**, run the full model for decode. Concretely (derived from layer counts; Appendix A confirms layers 25–36 for Qwen3-VL):

| Model | Layers | Skipped at prefill | Ratio |
|---|---|---|---|
| Llama-3.1-8B-Instruct | 32 | 10 (layers 23–32) | **31.25%** |
| Qwen3-VL-8B-Instruct | 36 | 12 (layers 25–36) | **33.3%** |
| Gemma-3-12B-It | 48 | 16 (layers 33–48) | **33.3%** |

The ratio was "empirically selected"; the sweep is in § Pruning-ratio sensitivity. Layer selection is not per-input and not learned — the virtual-gate analysis motivates the rule, it is not consulted at run time.

### 4. The KV problem and the "independent KV projections"

If layer *l* is skipped at prefill, its K_l, V_l for the prompt tokens do not exist — yet decode runs the full model and layer *l*'s attention must attend over the prompt. Removing those entries entirely is catastrophic (ablation below: **2.05% GSM8K, 1.18% HotpotQA**), because the deep layers can then only attend to the last prompt token and the generated tokens.

The fix is to **decouple the K/V projections from the block**. For every skipped layer, prefill still computes

  k_l = RoPE(LN(x_l) W^K),  v_l = LN(x_l) W^V

and stores them, then passes x_l through unchanged (x̂_{l+1} = x_l). Two things to be precise about, since the phrase "independent KV projections" invites over-reading:

- **They are the layer's own pretrained W^K, W^V and its own pre-attention norm.** Nothing is trained, distilled, or calibrated for this path; there are no new parameters. "Independent" means *executed independently of the attention/FFN blocks*, not *separately learned*. The paper's own summary: POP "requires no model retraining or specialized hardware or kernels".
- **Every skipped layer projects the same hidden state.** Because the skipped block is contiguous and each skip is an identity, layers 2L/3+1 … L all see the output of the last executed layer as their input. Each applies its own LN and projections to it, so the cached K/V differ per layer but derive from one representation rather than from *L*/3 progressively refined ones.

Cost: the paper states the K/V projections are **< 5% of the full attention + FFN cost** for Llama-3, Qwen-3 and Gemma-3, so almost all of the skipped layers' FLOPs are actually saved.

**Representation mismatch (Appendix A).** The obvious worry is that deep layers were trained on x_{l+1}-shaped inputs and now receive x_{2L/3}. Measured on Qwen3-VL, layers 25–36, WizardLM data, cosine similarity to the full model: hidden states drift from 1.0 down to **0.71**; cached **value states drift to 0.46**; but the **decode-stage attention output stays above 0.96** at every layer. The authors' reading: attention's weighted aggregation over the context smooths per-token drift, so the deep layers are "functionally robust" to inputs they were never trained on — and the virtual-gate score already captured that (low gradient variance ⇔ loss insensitive to the mismatch).

### 5. Boundary handling — the first generated token

Standard prefill processes x_{1:N} and its last position predicts x_{N+1}. But the deep layers are precisely what next-token prediction needs, so if x_N goes through the pruned stack the *first generated token* is degraded and the error compounds. POP therefore **redefines the boundary**: pruned prefill covers x_{1:N−1}; **x_N is processed as the first decode step with the full model.** This is one extra full-depth token per request. It is also why short prompts gain little (below).

## Results

### Accuracy (Table 1; 0-shot likelihood for the four common-sense tasks, greedy generation for the rest; A100 80 GB, HF Transformers)

Baselines were set to "comparable FLOPs reduction during prefill": Wanda 30% (unstructured), SliceGPT 25%, ShortGPT 25%. POP prunes *more* (31–33%) than the structured baselines (25%).

**Llama-3.1-8B-Instruct** (text only; 8 tasks)

| Method | MMLU | HellaSwag | WinoG | PIQA | GSM8K | HumanEval | MultiFieldQA | HotpotQA | **Avg** |
|---|---|---|---|---|---|---|---|---|---|
| Full | 68.33 | 79.50 | 74.40 | 81.12 | 79.68 | 68.29 | 54.57 | 55.66 | **70.19** |
| Wanda 30% (unstructured) | 65.87 | 78.96 | 74.59 | 80.74 | 76.42 | 65.84 | 52.80 | 53.03 | 68.53 |
| SliceGPT 25% | 34.97 | 51.19 | 66.54 | 63.87 | **0.91** | **0.00** | 12.35 | 8.71 | 29.82 |
| ShortGPT 25% | 65.80 | 61.93 | 69.77 | 70.51 | **0.38** | **0.00** | 6.80 | 3.81 | 34.88 |
| **POP 31.25%** | 67.43 | 78.29 | 73.40 | 80.36 | **77.26** | **64.63** | 52.88 | 53.48 | **68.47** |

POP vs full: −1.72 avg points. GSM8K retains 77.26/79.68 = **97.0%** (paper: 97.00%); HumanEval retains 64.63/68.29 = **94.6%** (paper says 95.64% — arithmetic error, see below).

**Qwen3-VL-8B-Instruct** (12 tasks incl. MMMU, RealWorldQA, TextVQA, ScreenSpot)

| Method | MMLU | HellaSwag | WinoG | PIQA | GSM8K | HumanEval | MFQA | HotpotQA | MMMU | RWQA | TextVQA | ScreenSpot | **Avg** |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Full | 74.95 | 76.60 | 73.72 | 79.92 | 81.50 | 92.07 | 53.53 | 65.49 | 51.33 | 69.67 | 82.24 | 87.03 | **74.00** |
| Wanda 30% | 73.78 | 75.22 | 72.45 | 80.47 | 83.32 | 90.85 | 52.87 | 63.19 | 52.00 | 67.45 | 81.08 | 85.22 | 73.16 |
| SliceGPT 25% | 39.16 | 44.50 | 57.93 | 67.25 | 13.95 | 17.68 | 40.76 | 38.33 | 28.00 | 32.55 | 13.54 | **0.24** | 32.82 |
| ShortGPT 25% | 33.85 | 48.24 | 61.56 | 64.96 | 0.83 | 0.00 | 21.44 | 16.37 | 32.22 | 53.07 | 33.69 | **0.86** | 30.59 |
| **POP 33.3%** | 75.05 | 76.44 | 73.88 | 80.14 | 80.21 | 89.63 | 52.34 | 63.13 | 50.67 | 69.28 | 80.73 | 86.40 | **73.16** |

POP vs full: −0.84 avg, tied with Wanda. Largest single drops: HumanEval −2.44, HotpotQA −2.36. MMMU 50.67 vs 51.33 and ScreenSpot 86.40 vs 87.03 are the paper's "near-lossless" multimodal examples — verified.

**Gemma-3-12B-It** (12 tasks)

| Method | MMLU | HellaSwag | WinoG | PIQA | GSM8K | HumanEval | MFQA | HotpotQA | MMMU | RWQA | TextVQA | ScreenSpot | **Avg** |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Full | 71.46 | 81.96 | 74.35 | 78.07 | 73.62 | 82.32 | 55.90 | 59.62 | 46.78 | 54.64 | 67.02 | 11.08 | **63.07** |
| Wanda 30% | 69.70 | 80.82 | 73.64 | 77.42 | 75.13 | 83.54 | 55.28 | 58.78 | 45.89 | 55.29 | 64.67 | 10.38 | 62.55 |
| SliceGPT 25% | 22.95 | 34.12 | 54.14 | 55.93 | 1.67 | 0.00 | 10.83 | 4.18 | 25.56 | 5.23 | 2.59 | 0.24 | 18.12 |
| ShortGPT 25% | 23.81 | 30.32 | 48.70 | 53.70 | 0.91 | 0.00 | 1.58 | 0.34 | 25.00 | 0.39 | 0.00 | 0.24 | 15.42 |
| **POP 33.3%** | 71.37 | 81.96 | 74.59 | 79.76 | 73.16 | 81.10 | 57.33 | 59.11 | 46.78 | 55.42 | 63.71 | 11.08 | **62.95** |

POP vs full: **−0.12 avg**; five tasks tie or improve (PIQA +1.69, MultiFieldQA +1.43); TextVQA −3.31 is the one real loss. (Gemma-3's 11.08 on ScreenSpot is the dense model's own floor, not a pruning effect.) All twelve averages in the three tables recompute correctly from the cells.

**Layer-pruning baselines with numbers: ShortGPT only.** LaCo and SLEB are cited in Related Work as layer-wise structured pruners but are **not** in any table; the only layer-pruning comparison is ShortGPT at 25%, which scores 34.88 / 30.59 / 15.42 average against POP's 68.47 / 73.16 / 62.95. The paper's read is that any whole-model layer removal destroys generation (0–0.9% GSM8K, 0% HumanEval across all three models) while keeping likelihood-ranking benchmarks partly intact — the same layers POP skips, but only in one stage.

### Prefill speedup (Table 2; TTFT on A100, batch 8, relative to the full model)

| Llama-3.1-8B-Instruct — tokens | 32 | 128 | 512 | 2048 |
|---|---|---|---|---|
| Wanda 30% | 1.00 | 1.00 | 1.00 | 1.00 |
| SliceGPT 25% | 1.22 | 1.31 | 1.29 | 1.31 |
| ShortGPT 25% | **1.30** | 1.29 | 1.31 | 1.30 |
| **POP 31.25%** | 1.22 | 1.27 | 1.34 | **1.36** |

| Gemma-3-12B-It — tokens | 32 | 128 | 512 | 2048 |
|---|---|---|---|---|
| SliceGPT 25% | 1.10 | 1.29 | 1.27 | 1.29 |
| ShortGPT 25% | **1.25** | 1.29 | 1.31 | 1.31 |
| **POP 33.3%** | **1.02** | 1.27 | 1.34 | **1.37** |

| Qwen3-VL-8B-Instruct — image | 640×480 | 1280×720 | 1920×1080 | 2560×1440 |
|---|---|---|---|---|
| SliceGPT 25% | 1.14 | 1.16 | 1.15 | 1.14 |
| ShortGPT 25% | 1.18 | 1.17 | 1.15 | 1.13 |
| **POP 33.3%** | **1.19** | **1.19** | 1.18 | 1.16 |

The **1.37× headline checks**: Gemma-3 at 2,048 tokens (Table 2) and Qwen3-VL at 1,024 text tokens / batch 4 (Table 4). Three readings the paper makes and the numbers support:

- **Short prompts gain little** (1.22× Llama, **1.02× Gemma at 32 tokens**) — short prefill is memory-bound on weight reads, and the boundary rule forces one full-model pass regardless, so the skipped layers' weights are read anyway. ShortGPT, which prunes the weights out of both stages, beats POP at 32 tokens on both models.
- **The speedup converges to the FLOP ratio at long inputs.** *Derived:* skipping fraction *p* of layers with the <5% K/V residue gives an ideal ≈ 1/(1 − 0.95·p) = **1.42×** at 31.25% and **1.46×** at 33.3%; POP reaches 1.36–1.37× = 94–96% of ideal at 2,048 tokens. ShortGPT's ideal at 25% is 1.33× and it measures 1.30–1.31×. So POP's *speed* edge over ShortGPT is entirely that it removes more layers; its *accuracy* edge is the stage split.
- **Vision inputs get 1.16–1.19×**, below text. The paper does not say why; the plausible reason (this wiki's inference, not the paper's) is that the vision encoder's share of TTFT is not pruned.
- Wanda's 1.00× everywhere is the paper's point that unstructured sparsity needs sparse kernels to be worth anything on an A100.

### Ablations (Table 3; Qwen3-VL-8B-Instruct)

| Variant | GSM8K | HotpotQA |
|---|---|---|
| Full model | 81.50 | 65.49 |
| **POP (last third)** | **80.21** | **63.13** |
| Shallow pruning (first third) | 0.15 | 0.00 |
| Interleaved (every 3rd layer) | 56.48 | 6.81 |
| w/o independent KV projections | 2.05 | 1.18 |
| w/o boundary handling | 77.33 | **11.45** |

Three things this settles. Redundancy is *located*, not uniform — pruning shallow layers at prefill is total collapse. The KV projections are load-bearing, not an optimization. And the boundary rule matters far more for long-context QA (−51.7 points on HotpotQA) than for GSM8K (−2.9): a wrong first token after a long document derails the answer.

### Pruning-ratio sensitivity (Table 4; Qwen3-VL-8B-Instruct, 1,024 tokens, batch 4)

| Ratio | Speedup | GSM8K | HotpotQA |
|---|---|---|---|
| 0% | 1.00× | 81.50 | 65.49 |
| 20% | 1.19× | **83.09** | 65.46 |
| 25% | 1.25× | 82.34 | **65.81** |
| **33% (default)** | **1.37×** | 80.21 | 63.13 |
| 40% | 1.46× | 80.82 | 61.69 |
| 50% | 1.67× | 78.54 | **34.69** |
| 60% | 1.96× | 38.51 | 5.45 |

At 20–25% the pruned model *beats* the full one on GSM8K (83.09 vs 81.50; the authors call it regularization). GSM8K is robust to 50%; HotpotQA halves at 50% and collapses at 60%. The text says the sharp decline comes "beyond 50%", but HotpotQA has already lost 27 points *at* 50% — the knee is at 40→50%, so 40% (1.46×) is the last defensible point for long-context work and 33% is the conservative default.

### Composition with sparse attention and prompt compression (Appendix B, Table 5; Llama-3.1-8B-Instruct, batch 1, 32K input)

| Method | TTFT speedup | HotpotQA | MultiFieldQA |
|---|---|---|---|
| Full | 1.00× | 57.83 | 56.00 |
| FlexPrefill γ=0.95 | 1.13× | 56.39 | 54.41 |
| FlexPrefill γ=0.8 | 1.22× | 47.79 | 48.55 |
| **FlexPrefill γ=0.95 + POP** | **1.54×** | 54.34 | 53.42 |
| LLMLingua-2 (0.4) | 1.37× | 56.11 | 42.49 |
| LLMLingua-2 (0.3) | 1.53× | 50.92 | 39.90 |
| **LLMLingua-2 (0.4) + POP** | **1.56×** | 51.74 | 42.31 |

POP is depth-axis, [[sparse-attention]] is attention-entry-axis, [[prompt-compression]] is sequence-axis, so they stack: a conservative FlexPrefill plus POP gives 1.54× at −3.5 HotpotQA, where FlexPrefill alone pushed to 1.22× costs −10.0. **POP alone at 32K / batch 1 is not reported in this table**, so its own contribution at that length has to be inferred (≈1.36× from Table 2 at 2K, batch 8 — a different setting).

## Is it training-free? — plainly, yes

No weights are modified, added, trained, or distilled. The deployed rule is "skip the last third of layers for tokens 1…N−1; run their K/V projections; run x_N and everything after with the full model". The virtual-gate importance analysis needs gradients on **200 calibration samples** (WizardLM-V2-196K or LLaVA-Instruct-150K, one forward-backward pass each, with targets sampled from the model) — but that analysis was used to *discover* the rule, and the rule is static, so a deployment does not even need to rerun it. It is closer to a serving-engine flag than to a compression method.

## Claims that did not check out, or need qualification

- **"HumanEval retains 95.64%"** (§ 4.2): 64.63 / 68.29 = **94.64%**. Arithmetic slip; the GSM8K 97.00% figure is correct.
- **"SliceGPT degrades ScreenSpot from 87.03% to 0.86%"** (§ 4.2): Table 1 gives SliceGPT **0.24** and ShortGPT **0.86** — the sentence attributes ShortGPT's number to SliceGPT. Either way both are collapsed.
- **"Pushing the ratio beyond 50% leads to a sharp decline"**: HotpotQA is already 34.69 *at* 50% (from 61.69 at 40%). The decline starts at 50%, not beyond it.
- **Full-model HotpotQA / MultiFieldQA differ between Table 1 (55.66 / 54.57) and Table 5 (57.83 / 56.00)** for the same Llama-3.1-8B-Instruct with greedy decoding in both. Different context windows or evaluation runs are the likely cause; not explained.
- **LaCo and SLEB have no numbers** despite being named alongside ShortGPT; the layer-pruning comparison is ShortGPT only.
- **Fig. 1 (the asymmetry evidence) is qualitative** — no importance values are tabulated; the claim that final-layer decode importance "exceeds the visualization range" cannot be quantified from the paper.
- **Speedup and accuracy are measured in different configurations**: Table 1 accuracy is per-benchmark; Table 2 speed is batch 8 at fixed lengths; Table 4 is batch 4 at 1,024 tokens; Table 5 is batch 1 at 32K. No single row pairs a benchmark score with the latency at that benchmark's prompt length.
- Everything else verified: all three Table 1 averages recompute from the cells; 31.25% = 10/32; 33.3% = 12/36 and 16/48; layers 25–36 for Qwen3-VL; 1.36×/1.37× at 2,048; 1.02×/1.22× at 32; 1.16–1.19× on images; ablation cells; Table 4 cells; Appendix A's 0.71 / 0.46 / >0.96; 200 calibration samples; A100 80 GB; <5% K/V cost.

## Limitations

- **No memory saving, no decode speedup.** The full model must be resident for decode, so peak VRAM is unchanged and every generated token costs the same as before. For generation-heavy workloads the end-to-end gain is small; the paper is explicit that POP is for "compute-bound rather than capacity-bound scenarios".
- **Ceiling is ~1.4×.** Skipping a third of the depth cannot give more than ~1.46× on prefill FLOPs, and 40% (1.46× measured) is already at the HotpotQA knee.
- **Short prompts and vision inputs gain little** (1.02–1.22× at 32 tokens; 1.16–1.19× on images).
- **One static rule for every input.** Layer selection is not adaptive; the importance analysis is only run offline on 200 samples. Whether a per-task or per-length ratio would be better is not explored.
- **Same-representation K/V for all skipped layers** — value-state similarity falls to 0.46 vs the full model. The attention output stays >0.96 on the measured model, but this was checked on one model (Qwen3-VL) with one dataset.
- **Implementation is a monolithic HF Transformers pipeline**; the disaggregated prefill/decode systems where a stage-specific model is most natural ([[distserve-zhong-2024]], [[splitwise-patel-2023]]) are named as future work.
- **Architecture specifics are not discussed** — e.g. whether Gemma-3's interleaved local/global attention layers or Qwen3-VL's vision encoder interact with a contiguous last-third skip.
- Three model families, 8B–12B only; no result on models >12B, MoE models, or reasoning models with long decode.

## Related

- [[fast-forward-predictive-ffn-sparsity-prefill-gautam-2026]] — the sibling prefill-only paper, same week (30 Jan vs 3 Feb 2026), skipping **FFN width** (50% of neurons per 128-token block, learned predictor + compensator) where POP skips **depth**. Trained vs training-free; 1.44× vs 1.37× at 2K on an 8B; −6% relative LongBench vs −0.1 to −1.7 avg points. Orthogonal axes, never tested together.
- [[methods-to-speed-up-prompt-processing]] · [[sparse-prefill]] — the wiki's prefill taxonomy is sequence-axis (tokens) and attention-axis (entries). POP is a **model-axis** entry that keeps every token and every KV slot; it composes with both (Appendix B).
- [[kv-cache]] — the whole difficulty is that decode needs a *complete* cache; the K/V-projection-only path is the paper's answer, and its representation drift (Appendix A) is the cost.
- [[layer-skipping]] · [[structured-pruning]] — **no pages yet.** POP is a stage-conditioned layer skip; ShortGPT/LaCo/SLEB/SliceGPT/LLM-Pruner are the stage-agnostic structured pruners it is defined against. [[recurrent-depth-transformer]] is the nearest existing page on treating depth as variable.
- [[discovering-the-gems-in-early-layers-shi-2024]] · [[claa-cross-layer-attention-aggregation-mcdanel-2026]] — the token-side cousins: they also find that early/middle layers already carry what is needed for context selection. POP's Fig. 1 says the same for context *encoding*.
- [[token-pruning]] · [[prompt-compression]] · [[llmlingua-2-pan-2024]] · [[sparse-attention]] · [[minference-dynamic-sparse-attention-jiang-2024]] — the methods Appendix B composes with (LLMLingua-2, FlexPrefill).
- [[arithmetic-intensity]] · [[llm-inference-roofline-yuan-2024]] — why the gain vanishes at 32 tokens: memory-bound prefill is not helped by skipping FLOPs, and the boundary token reads every weight anyway.
- [[distserve-zhong-2024]] · [[splitwise-patel-2023]] · [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]] — disaggregated serving, where "a different model for prefill" is a deployment primitive rather than a hack.
- [[chunked-prefill]] · [[sarathi-chunked-prefill-agrawal-2023]] — orthogonal scheduling axis; POP reduces the prefill work chunked prefill schedules.
- [[activation-sparsity]] · [[weight-streaming]] — the neuron-axis sparsity that Fast Forward, [[llm-in-a-flash-alizadeh-2023]] and [[neuroprefetcher-delta-prefetching-dhar-2026]] exploit; POP uses none of it.
- [[llama]] · [[qwen]] · [[gemma]] (no page) · [[mmlu]] · [[humaneval]] · [[longbench]] (MultiFieldQA/HotpotQA are LongBench tasks) · [[nvidia]] (A100).
- [[grouped-query-attention]] — all three models use GQA; the K/V projections POP keeps are the small GQA ones, which is part of why they cost <5%.

## References

- [arXiv:2602.03295](https://arxiv.org/abs/2602.03295) (v2, 16 Apr 2026).
- ShortGPT: Men et al. 2025 — cosine-similarity layer removal; the layer-pruning baseline with numbers. LaCo: Yang et al. 2024b; SLEB: Song et al. 2024 — cited, not compared.
- SliceGPT: Ashkboos et al., ICLR 2024 — PCA row/column deletion baseline. Wanda: Sun et al. 2024 — unstructured baseline. SparseGPT: Frantar & Alistarh, ICML 2023; LLM-Pruner: Ma et al. 2023.
- Molchanov et al. 2019; LeCun et al. 1989 (Optimal Brain Damage); Kunstner et al., NeurIPS 2019 — the Taylor/Fisher importance estimator lineage behind the virtual gate.
- FlexPrefill: Lai et al., ICLR 2025; LLMLingua-2: Pan et al. 2024 — Appendix B composition partners. MInference: Jiang et al., NeurIPS 2024; MMInference: Li et al. 2025.
- DistServe: Zhong et al. 2024; Splitwise: Patel et al. 2024 — disaggregated serving.
- Token pruning cited: LLMLingua (Jiang 2023), PyramidInfer (Yang 2024a), DAC (Zhao 2025), FastV (Chen 2024), DART (Wen 2025). Decode-side sparse KV: Quest, PQCache, MagicPIG.
- Models: Llama 3.1 (Team 2024); Qwen3-VL technical report, [arXiv:2511.21631](https://arxiv.org/abs/2511.21631); Gemma 3 (Team 2025). Evaluation: LM-Evaluation-Harness, LongBench (Bai et al., ACL 2024), LMMs-Eval.
