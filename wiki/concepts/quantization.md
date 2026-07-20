---
title: Quantization
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - gptq-frantar-2022.md
  - awq-lin-2023.md
  - smoothquant-xiao-2022.md
  - llm-int8-dettmers-2022.md
  - qlora-dettmers-2023.md
  - quarot-ashkboos-2024.md
  - bitnet-b158-ma-2024.md
status: complete
importance: high
---

# Quantization

**Quantization** stores a model's weights (and sometimes activations and the [[kv-cache|KV cache]]) at lower numeric precision than the FP16/BF16 they were trained in — typically 8, 4, or fewer bits. It is the dominant technique for making large models fit and run on constrained hardware.

This page covers **methods**. The effects are split across three pages:
- [[quantization-performance]] — memory, and why prefill (`pp`) and decode (`tg`) respond oppositely
- [[quantization-quality]] — perplexity and capability preservation
- [[quantization-safety]] — alignment degradation, plus [[quantization-conditioned-backdoor|quantization as an attack vector]]

## The four axes

**1. Weight-only vs weight+activation.** The single most consequential distinction.

| | W-only (W4A16, W8A16) | W+A (W8A8, W4A4) |
|---|---|---|
| Shrinks | memory + weight-load bandwidth | memory **and** compute |
| Kernel | dequantize to FP16, then FP16 matmul | native INT8/INT4/FP8 tensor-core GEMM |
| Wins when | **bandwidth-bound**: single-stream, small batch, local | **compute-bound**: large-batch serving, prefill |
| Examples | GPTQ, AWQ, SqueezeLLM, QuIP#, AQLM, NF4, GGUF, EXL2 | SmoothQuant, LLM.int8(), QuaRot, SpinQuant, FP8 |

**2. Granularity** (coarse → fine): per-tensor → per-channel/per-token → per-group. **Group size 128** is the near-universal default. Finer groups resist outliers better but cost metadata: [[qlora-dettmers-2023|QLoRA]] quantifies it exactly — FP32 scales at block 64 cost **0.5 bits/param**, which is why every fine-grained scheme also quantizes its own scales.

**3. Symmetric vs asymmetric.** Symmetric (`x ≈ s·q`) folds cleanly out of a GEMM; asymmetric (`x ≈ s·(q−z)`, with an integer zero-point) fits skewed distributions far better but adds a cross-term to integer matmuls. Hence weight-side schemes use zero-points freely — dequantization happens before the FP16 matmul anyway — while activation-side integer schemes stay symmetric.

**4. PTQ vs QAT.** A useful dividing test: **does the method update the model's own weights via gradients, or only auxiliary quantization parameters?** By that test GPTQ/AWQ/SmoothQuant/HQQ/SpinQuant are post-training; QuIP#/AQLM/LLM-QAT/BitNet are on the training side. The boundary has genuinely blurred — QuIP# and AQLM include fine-tuning passes inside nominally PTQ pipelines.

## Why outliers are *the* central obstacle

This is the through-line of the entire field.

1. **Discovery** — [[llm-int8-dettmers-2022|LLM.int8()]] found outlier features emerge as a **phase transition at ~6.7B parameters**: at that scale, **150,000 outliers per sequence concentrated in only 6 feature dimensions**. Zeroing those 6 dimensions (0.1% of features) degrades perplexity by **600–1000%**; zeroing the same number of random features costs ~0.1%.
2. **Characterization** — [[massive-activations-attention-sinks-sun-2026|Massive Activations]] shows these values are ~10⁵× larger than typical, sit at fixed locations, are largely **input-invariant**, and function as *indispensable bias terms* — connecting them to attention sinks.
3. **Diagnosis** — "Quantizable Transformers" ([arXiv:2306.12929](https://arxiv.org/abs/2306.12929)) argues outliers arise because **softmax cannot output exactly zero**: a head that wants to attend to *nothing* must drive activations to extremes to starve the softmax.
4. **Why they're hard to scale away**: for activations the outliers live in *channels* — but the channel axis **is** the GEMM's reduction axis, so a per-channel activation scale cannot factor out of the inner product. [[smoothquant-xiao-2022|SmoothQuant]] makes exactly this point. Outliers sit on the one axis you cannot cheaply rescale, and that asymmetry is the reason SmoothQuant, QuaRot and SpinQuant all exist.

### Four families of workaround

| Family | Idea | Methods |
|---|---|---|
| **Keep in high precision** | isolate outliers, run them FP16 | LLM.int8(), SpQR, SqueezeLLM (0.45% sparse) |
| **Move them** | equivalent per-channel rescaling migrates difficulty into weights | [[smoothquant-xiao-2022\|SmoothQuant]], AWQ, OmniQuant |
| **Rotate them away** | orthogonal/Hadamard transform makes the hidden state *incoherent*, spreading outlier energy evenly so no channel is special | QuIP → [[quarot-ashkboos-2024\|QuaRot]] → SpinQuant |
| **Never create them** | architectural fix, or train natively low-precision | Quantizable Transformers, [[bitnet-b158-ma-2024\|BitNet b1.58]], MXFP4 QAT |

The **rotation family** is arguably the key conceptual advance: rather than special-casing outliers, change the basis so they *cannot exist* — a random orthogonal transform spreads a vector's energy evenly with high probability. Both QuaRot and SpinQuant exploit the **computational invariance** of the residual stream (rotate it, fold the inverse into adjacent weight matrices, output unchanged). This is what finally made W4A4 work with *no* retained-precision channels.

## Methods

### Weight-only PTQ

| Method | Mechanism | Bits | Calibration |
|---|---|---|---|
| **[[gptq-frantar-2022\|GPTQ]]** | layer-wise second-order (inverse-Hessian) error compensation: quantize one column, update the rest to absorb the error | 4/3/2 | 128 × 2048 tok (C4) |
| **[[awq-lin-2023\|AWQ]]** | protect ~1% *salient* channels — identified by **activation** magnitude, not weight magnitude — via equivalent per-channel scaling | 3/4, g128 | tiny (16 seqs suffice) |
| **SqueezeLLM** | weighted k-means codebook using the Fisher diagonal as a Hessian proxy + 0.45% FP16 sparse outliers | 3/4 | 100 samples |
| **QuIP#** | Hadamard incoherence + **E₈ lattice** vector codebooks + fine-tuning | 2 | ~25M tokens |
| **AQLM** | additive quantization — a weight group as a *sum* of learned codebook vectors | 2–3 | 8M tokens |
| **HQQ** | ℓp (p<1) loss solved by half-quadratic splitting; optimizes only the zero-point | 2–8 | **none** |

### Weight+activation

- **[[smoothquant-xiao-2022|SmoothQuant]]** — migrates activation outliers into weights so both are quantizable. W8A8, unlocking INT8 GEMM. Migration strength α = 0.5 works for OPT/BLOOM.
- **[[quarot-ashkboos-2024|QuaRot]]** — Hadamard rotations give **W4A4KV4 end-to-end** with no high-precision channels.
- **SpinQuant** ([arXiv:2405.16406](https://arxiv.org/abs/2405.16406)) — *learns* the rotation on the Stiefel manifold via Cayley SGD, after observing random rotations vary by up to 13 points downstream.
- **QServe** ([arXiv:2405.04532](https://arxiv.org/abs/2405.04532)) — W4A8KV4; the pragmatic finding that W4A4 is often *slower* in practice than W4A8 due to dequantization overhead.

### bitsandbytes

- **[[llm-int8-dettmers-2022|LLM.int8()]]** — vector-wise INT8 + a mixed-precision escape hatch keeping outlier dimensions in FP16. Calibration-free.
- **[[qlora-dettmers-2023|NF4 / QLoRA]]** — **NormalFloat4**, a quantile grid built from the normal distribution (weights are ~zero-centered normal), made asymmetric so zero is exactly representable. Double Quantization cuts scale overhead 0.5 → 0.127 bits/param. Enables finetuning a **65B model on one 48GB GPU**.

### Formats without papers

- **[[gguf|GGUF k-quants]]** ([[llama-cpp]]) — block-wise scaling with **super-blocks**: per-block scales are themselves quantized under a super-block FP16 scale, analogous to QLoRA's Double Quantization. `Q4_K` = 4.5 bpw. The `_S`/`_M`/`_L` suffixes are **tensor-assignment policies**, not different block formats — `Q4_K_M` is mostly Q4_K with Q6_K on sensitive tensors.
- **i-quants** (IQ2_XXS…) — E₈/D₄ lattice codebooks, explicitly crediting QuIP#. Require an **importance matrix** calibration pass; better at ≤3 bits but slower to decode.
- **EXL2** — GPTQ-based with **mixed bitrate per layer**, so you can target an arbitrary average bpw to exactly fill a VRAM budget.

> [!warning] Common misconception
> The "K" in llama.cpp **k-quants does not mean k-means**. The originating PR describes super-block scaling with no k-means at all; the actual etymology is undocumented.

### Low-precision formats

- **FP8** — E4M3 (weights/activations) and E5M2 (gradients). A *native tensor-core type* on Ada/Hopper/Blackwell, so it is a genuine compute speedup. [[deepseek-r1|DeepSeek]]-V3 trained a 671B model in FP8 using per-1×128-tile activation scaling.
- **MXFP4** — block floating point: 32 elements share one E8M0 power-of-two scale, E2M1 elements → **4.25 bits/param**. OpenAI's gpt-oss models were QAT'd to MXFP4 for their MoE weights (>90% of params), which is what fits 120B on one 80GB GPU.
- **NVFP4** — block 16 with an FP8 E4M3 scale. NVIDIA reports MXFP4 needs **~36% more tokens** to reach NVFP4's loss — a clean measurement of the block-size/scale-precision tradeoff.

### Extreme / ternary

**[[bitnet-b158-ma-2024|BitNet b1.58]]** constrains weights to **{−1, 0, +1}** (log₂3 = 1.58 bits) with 8-bit activations, making matmuls **addition-only**. Crucially it is **trained from scratch**, not converted — so it says nothing about quantizing an existing FP16 model to ternary, and the "1-bit LLM" branding is marketing-adjacent.

## The 4-bit consensus

Dettmers & Zettlemoyer's **k-bit inference scaling laws** ([arXiv:2212.09720](https://arxiv.org/abs/2212.09720), ICML 2023) established **4-bit as the near-universal accuracy-per-bit optimum** — the empirical basis for why Q4_K_M and INT4 are the defaults everywhere. See [[quantization-quality]] for where that breaks down.

> [!note] Cross-paper numbers are not comparable
> Calibration corpora differ (GPTQ→C4, AWQ/SmoothQuant→Pile, AQLM/QuIP#→RedPajama, QuaRot/SpinQuant→WikiText-2), and QuaRot/SpinQuant **calibrate on WikiText-2 train while evaluating WikiText-2 perplexity** — a same-distribution setup the others avoid. AQLM's table also cites QuIP#'s *no-fine-tuning* number against its own fine-tuned result. Treat leaderboard-style PPL comparisons across papers with suspicion.

## Related

- [[quantization-performance]] · [[quantization-quality]] · [[quantization-safety]] · [[quantization-conditioned-backdoor]]
- [[kv-cache]] · [[speculative-decoding]] · [[lora]] · [[knowledge-distillation]] — adjacent efficiency techniques.
- [[massive-activations-attention-sinks-sun-2026]] — the outlier phenomenon.

## References

- [[gptq-frantar-2022]] · [[awq-lin-2023]] · [[smoothquant-xiao-2022]] · [[llm-int8-dettmers-2022]] · [[qlora-dettmers-2023]] · [[quarot-ashkboos-2024]] · [[bitnet-b158-ma-2024]]
