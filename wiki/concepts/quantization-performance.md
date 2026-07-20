---
title: "Quantization Performance: Memory, Prefill (pp) and Decode (tg)"
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - llm-inference-roofline-yuan-2024.md
  - marlin-frantar-2024.md
  - give-me-bf16-or-give-me-death-kurtic-2024.md
  - which-quantization-kurt-2026.md
builds-on:
  - [[quantization]]
status: complete
importance: high
---

# Quantization Performance: Memory, Prefill (pp) and Decode (tg)

The single most important fact about [[quantization]] performance: **LLM inference has two phases with opposite bottlenecks, and quantization helps only one of them.**

- **Prefill / prompt processing (`pp`)** — compute-bound. Quantization helps little, and **weight-only quantization can make it slower**.
- **Decode / token generation (`tg`)** — memory-bandwidth-bound. Quantization gives a near-linear speedup.

Everything below follows from that asymmetry.

## The roofline argument

A kernel is compute-bound iff its arithmetic intensity `I = FLOPs / bytes moved` exceeds the hardware ridge point `I_ridge = peak_FLOPS / peak_bandwidth` (~200 OPs/byte for an A6000 at 155 TFLOPS / 768 GB/s).

Measured per-layer intensities for Llama-2-7B ([[llm-inference-roofline-yuan-2024|Yuan et al. 2024]]):

| Phase | Layer | `I` (OPs/byte) | Bound |
|---|---|---|---|
| **Prefill** | q/k/v/o_proj | **1,024** | compute |
| **Prefill** | gate/up/down_proj | **1,215** | compute |
| **Decode** | all projections | **1** | memory |
| **Decode** | attention ops | **0.99** | memory |

**A ~1000× gap in arithmetic intensity between two phases of the same model.**

- **Decode**: generating one token reads every weight exactly once and does ~2 FLOPs per weight, so `I ≈ 1` — roughly 200× below the ridge. The processor idles waiting on memory, and `tg_tokens_per_sec ≈ memory_bandwidth / model_bytes`. **Halve the bytes, roughly double tg.** Dequantization FLOPs hide under a memory pipe with 200× spare compute, so the speedup is nearly free.
- **Prefill**: processing S prompt tokens turns each matrix-vector into a GEMM — the same weight read serves S tokens, so `I ≈ S`. Far above the ridge, compute is the wall. Reducing weight bytes relieves a constraint that was **not binding**, while dequantization instructions add work to the critical path.

## The cleanest demonstration (llama.cpp, Apple Silicon)

LLaMA-7B via `llama-bench`, Metal backend ([llama.cpp discussion #4167](https://github.com/ggml-org/llama.cpp/discussions/4167)). Ratios derived from the published table:

| Chip | pp F16→Q4_0 | tg F16→Q4_0 | tg F16→Q8_0 |
|---|---|---|---|
| M1 Pro | **0.88× (12% slower)** | **2.86×** | 1.75× |
| M2 Ultra | **0.88×** | **2.30×** | 1.62× |
| M3 Max | 0.98× | **2.64×** | 1.70× |
| M4 Max | 0.96× | **2.63×** | 1.71× |

Same hardware, same model: 4-bit makes **prefill 2–12% slower** and **decode 2.3–2.9× faster**.

### Dequantization overhead is visible in bandwidth utilization

Achieved bandwidth (`tg × model_size`) falls monotonically as bit width drops:

| Chip | Peak | F16 | Q8_0 | Q4_0 |
|---|---|---|---|---|
| M1 Pro | 200 GB/s | 86% | 80% | **70%** |
| M3 Max | 400 GB/s | 85% | 77% | **63%** |
| M2 Ultra | 800 GB/s | 69% | 60% | **45%** |

This is why observed tg speedup (2.3–2.9×) undershoots the ideal size ratio (3.53×): lower precision means more ALU work per byte delivered. The effect is worst on the widest-bandwidth chip — more bandwidth to saturate, same per-byte dequant cost.

> [!note] On CPU, prefill is about kernel maturity, not physics
> [[which-quantization-kurt-2026|Kurt 2026]] (Llama-3.1-8B, dual Xeon 8488C) finds `tg128` spans 2.83 → 9.91 t/s (**3.5×**, cleanly correlated with file size) while `pp512` is **non-monotonic and uncorrelated with bit width**: Q4_0 is fastest at 97.35 t/s, but higher-precision Q5_1 manages only 45.98, and Q3_K_S is *28% slower than F16*. Formats with mature AVX-512 GEMM paths win; those with awkward unpacking lose. Cite as evidence that quantization **does not reliably help prefill** — not as a trend.

## Memory

```
bytes = params × bpw / 8
bpw   = n_bits + (scale_bits + zero_bits) / group_size
```

**Effective bits-per-weight always exceeds the nominal bit width**, for three stacked reasons:

1. **Block metadata** — the `/group_size` term. `Q4_0` = 4 + 16/32 = **4.5** bpw. GPTQ/AWQ at group-128 symmetric = 4 + 16/128 = **4.125** (this is where Marlin's "ideal 3.87×" comes from: 16/4.125).
2. **Mixed tensor assignment** — the dominant effect. `Q3_K_S/M/L` share an identical nominal 3.4375 bpw yet measure 3.64 / 4.00 / 4.30 effective; the suffix is purely a policy over which tensors get promoted.
3. **Unquantized tensors** — norms stay F32, and embeddings/output are held high. On Llama-3-8B the 128k-token vocab makes `token_embd` ~6.5% of the model on its own.

Measured, Llama-3.1-8B:

| Format | Size | Effective bpw | Nominal | Excess |
|---|---|---|---|---|
| F16 | 14.96 GiB | 16.00 | 16 | — |
| Q8_0 | 7.95 GiB | 8.50 | 8.5 | 0% |
| Q6_K | 6.14 GiB | 6.56 | 6.5625 | 0% |
| Q4_K_M | 4.58 GiB | **4.89** | 4.5 | +8.8% |
| Q3_K_M | 3.74 GiB | **4.00** | 3.4375 | +16.2% |
| Q2_K | 2.96 GiB | **3.17** | 2.5625 | +23.6% |

**Rule of thumb**: effective ≈ nominal +0.1–0.2 for ≥5-bit, +0.4 for Q4_K_M, +0.6 for 3-bit and below on large-vocab models.

### KV cache — a separate consumer that scales with context

```
kv_bytes = 2 × n_layers × n_kv_heads × head_dim × bytes_per_elem × n_tokens × batch
```

It does **not** depend on parameter count. [[grouped-query-attention|GQA]] is the biggest lever, [[kv-cache]] quantization the next.

| Model | FP16 per token | At full context |
|---|---|---|
| Llama-3-8B (GQA-8) | 128 KiB | **16.0 GiB @ 128k** |
| Llama-2-7B (MHA) | 512 KiB | **16.0 GiB @ 32k** |
| Llama-3-70B (GQA-8) | 320 KiB | **40.0 GiB @ 128k** |

The Llama-2-7B row is the headline: at 32k context the *cache exceeds the FP16 weights* (16.0 vs 12.55 GiB).

Counter-intuitively, **4-bit KV cache beats FP8 KV cache** on perplexity in ExLlamaV2's evaluation — a Hadamard rotation smooths outliers before quantizing (the [[quarot-ashkboos-2024|QuaRot]] trick), while FP8's fixed exponent/mantissa split wastes range. Research-grade: **KVQuant** ([arXiv:2401.18079](https://arxiv.org/abs/2401.18079)) hits <0.1 ppl degradation at 3-bit; **KIVI** ([arXiv:2402.02750](https://arxiv.org/abs/2402.02750)) is tuning-free at 2-bit, giving 2.6× less peak memory → 4× larger batch → **2.35–3.47× throughput**. Note KIVI's win is *indirect*: it comes from fitting more concurrent sequences, not from moving fewer bytes per token.

## Batch size is the unifying variable

W4A16 stores weights in 4-bit but dequantizes to FP16 in-register, so **FLOPs are unchanged** — only bytes shrink. As batch grows, `I` grows linearly and the kernel crosses the ridge, exposing the dequant work. [[marlin-frantar-2024|Marlin]] quantifies the crossover: **~3.9× at batch ≤16–32, decaying to 1.5× at batch 128.** Its design rule is the best one-line summary of the whole topic — modern GPUs have a FLOP-to-byte ratio of 100–200, so W4A16 wins exactly while you do **fewer than ~25–50 MACs per 4-bit weight**.

| Regime | `I` | Bound | Best scheme |
|---|---|---|---|
| Decode, batch 1 | ~1 | bandwidth | **W4A16** (near-linear) |
| Decode, batch 16–32 | ~16–32 | bandwidth | **W4A16** (~3.9×) |
| Decode, batch 64–128 | ~64–128 | crossing ridge | W4A16 → 1.5×; W8A8 pulls ahead |
| Decode, batch ≫128 | >200 | compute | **W8A8 / FP8** |
| Prefill, any | 114–1215 | compute | **FP8 / INT8 only** |

### Why W8A8 / FP8 is a different animal

Weight-only quantization never touches the compute roof. **W8A8 and FP8 lower the roof itself** by running on low-precision tensor cores (~2× FP16 throughput on A100 INT8 and H100 FP8). So **FP8 accelerates prefill; W4A16 does not** — Databricks measures ~30% TTFT improvement from FP8 on H100, and states plainly that "INT8 weight-only quantization provides little to no speedup because our computation remains in FP16."

## Throughput vs latency: the deployment rule

[[give-me-bf16-or-give-me-death-kurtic-2024|Kurtic et al. 2024]] (~500,000 evaluations across A6000/A100/H100, Llama-3.1 8B/70B/405B) turns the asymmetry into a deployment decision:

| Scheme | Compression | Best for |
|---|---|---|
| **W4A16-INT** | ~3.5× | **Single-stream / latency** — 2.4× average speedup |
| **W8A8-INT** | ~2× | **Server / throughput** (best on A100) — 1.8× |
| **W8A8-FP8** | ~2× | **Server / throughput** (best on H100) — 1.8× |

Single-stream is pure batch-1 decode, so weight bytes are the whole story and W4A16 wins at every model size — giving **6–7× cost reduction for 405B, letting it run on 4 GPUs instead of 16**. Batched serving crosses the ridge point, so only formats that lower the compute roof still help.

Production servers complicate this further: continuous batching runs compute-bound prefills and memory-bound decodes *simultaneously*, and chunked-prefill piggybacking ([SARATHI](https://arxiv.org/abs/2308.16369)) deliberately raises arithmetic intensity — eroding the weight-only advantage by design.

## Related

- [[quantization]] — the hub: methods and formats.
- [[quantization-quality]] — what accuracy costs these speedups.
- [[kv-cache]] · [[speculative-decoding]] — adjacent inference optimizations.

## References

- [[llm-inference-roofline-yuan-2024]] · [[marlin-frantar-2024]] · [[give-me-bf16-or-give-me-death-kurtic-2024]] · [[which-quantization-kurt-2026]]
