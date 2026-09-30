---
title: "BaseRT: Best-in-Class LLM Inference on Apple Silicon via Native Metal"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2607.00501"
authors:
  - Prabod Rathnayaka
  - Fabian Waschkowski
  - Lukas Wesemann
year: 2026
venue: "arXiv preprint (cs.CL), 2026-07-01 — Base Compute, Melbourne"
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# BaseRT: Best-in-Class LLM Inference on Apple Silicon via Native Metal

**Source**: [arXiv:2607.00501](https://arxiv.org/abs/2607.00501) (v1, 2026-07-01, CC BY 4.0). Three authors at Base Compute, Melbourne — the company that sells the runtime. Code: [basecompute/baseRT](https://github.com/basecompute/baseRT).

Ingested 2026-09-10 from the arXiv HTML full text. Every number below was re-derived from Tables 1–6 of the body; ratios in the prefill tables are computed here from the paper's absolute throughputs, since the paper prints only prose ranges for prefill. Repository facts are from the GitHub API and the `docs/` tree on the same day.

## Problem: framework overhead on Metal

The paper's thesis is that the two dominant [[apple-silicon|Apple Silicon]] runtimes carry overhead that a purpose-built [[metal|Metal]] runtime does not. It names the overheads only at the level of design principles, not measurements:

- **[[llama-cpp|llama.cpp]]** — *"originally developed as a cross-platform, CPU-first codebase and subsequently extended with a Metal backend. Its architecture reflects these origins."* No specific mechanism is identified.
- **[[mlx|MLX]]** — *"a general-purpose array framework whose lazy evaluation and framework-level scheduling introduce overhead not present in a dedicated inference runtime."*

What BaseRT's design section (§3) actually targets is more concrete, and is all **CPU-side, per-token, decode-path** overhead:

1. **Per-operator scheduling, allocation, and graph-evaluation work** on the decode hot path — replaced by a "low-overhead command dispatch mechanism."
2. **Per-token CPU–GPU synchronisation** — a decode scheduling scheme *"amortises CPU–GPU synchronisation across multiple generated tokens."*
3. **One kernel launch plus one global-memory round-trip per un-fused operator** — removed by fusing operator sequences along the attention path and inside the FFN block; *"particularly impactful during decode, where individual kernel runtimes are short and launch overhead is proportionally large."*
4. **Materialised dequantised weights** — every matmul kernel dequantises in its inner loop, so memory traffic shrinks by the compression ratio.
5. **Allocator contention** — a zero-allocation decode loop: residual, attention, FFN scratch, logits, token buffers and the whole [[kv-cache]] (to max context) are pre-allocated at load time; even error handling uses a static thread-local buffer.

The paper does **not** say "command-buffer granularity" or "memory copies," and it presents no profiler breakdown or ablation attributing the measured speedup to any one of these five. Unified memory is invoked as "unified memory-aware data layout" and a KV layout "chosen to maximise coalesced GPU memory access" — again a principle, not a measured effect.

## Architecture

- **C++ runtime directly on Metal**, no MLX / PyTorch / CoreML / MPSGraph dependency; C API with Python, Node, Rust and Swift bindings; models in a native `.base` weight format.
- **Data-driven architecture descriptors** — activation/norm variants, MoE routing, attention and positional-encoding conventions are data, so the hot path never branches on architecture. Supported families at submission: LLaMA, Qwen3, [[gemma|Gemma]], Whisper, BERT.
- **Hand-written Metal shader library** across matmul, attention, norm, RoPE, embedding, activation and sampling. Separate **GEMV (decode, M=1)** and **GEMM (prefill, M>1)** kernels per quantisation format, 2- to 16-bit; eight formats total (Q2, Q3, Q4, Q5, Q6, Q8, BF16, FP16).
- **Hardware-adaptive kernel selection** — a per-chip configuration keyed on GPU core count and family (M1 through M5) sets threadgroup sizes, selection thresholds and fusion decisions.
- **Prefill** — tiled GEMM on `simdgroup_matrix` intrinsics with tile geometry tuned per sequence-length regime; [[chunked-prefill]] with a bounded chunk size to cap scratch memory; a FlashAttention-style tiled online-softmax kernel with O(n) memory. The paper states the standard [[arithmetic-intensity]] framing: prefill is compute-bound, decode memory-bound.

## Setup

| | M4 Pro (primary) | M3 base |
|---|---|---|
| GPU cores | 16 | 8 |
| Unified memory | 24 GB | 16 GB |
| Bandwidth | 273 GB/s | 100 GB/s |

Six models: dense **Qwen3-0.6B, Llama-3.2-1B, Llama-3.2-3B, Gemma-4-E2B**; [[mixture-of-experts|MoE]] **Gemma-4-26B-A4B, Qwen3-30B-A3B**. Q4 and Q8 for dense; **Q4 only for the MoE models** because 20B+ at Q8 exceeds 24 GB. Metrics are `llama-bench` style — `pp128…pp2048` and `tg128` (see [[quantization-performance]] for the vocabulary). Baselines: llama.cpp commit b9630 (June 2026), Metal backend, defaults; MLX v0.31.2 / mlx-lm defaults; uzu (Mirai) at matching bit widths. Standard deviations are omitted from tables, stated as **<3% decode, <10% prefill** — which matters below, because many prefill deltas are smaller than that.

## Decode (tg128), M4 Pro — Table 1

| Model | Quant | BaseRT | llama.cpp | ratio | MLX | ratio |
|---|---|---|---|---|---|---|
| Qwen3-0.6B | Q4 | **464.5** | 297.4 | **1.56×** | 343.6 | **1.35×** |
| | Q8 | 321.2 | 219.8 | 1.46× | 255.3 | 1.26× |
| Llama-3.2-1B | Q4 | 295.4 | 230.4 | 1.28× | 257.8 | 1.15× |
| | Q8 | 183.8 | 160.7 | 1.14× | 159.2 | 1.15× |
| Llama-3.2-3B | Q4 | 117.3 | 102.4 | 1.15× | 112.1 | 1.05× |
| | Q8 | 70.9 | 65.1 | 1.09× | 65.5 | 1.08× |
| Gemma-4-E2B | Q4 | 127.7 | 107.0 | 1.19× | — | — |
| | Q8 | 84.5 | 59.5 | 1.42× | — | — |
| Gemma-4-26B-A4B (MoE) | Q4 | 62.2 | 58.0 | 1.07× | 69.3 | **0.90×** |
| Qwen3-30B-A3B (MoE) | Q4 | 84.1 | 80.7 | 1.04× | 83.1 | 1.01× |

Gemma-4-E2B has no MLX baseline ("not supported in MLX"). The abstract's headline numbers — **1.56× over llama.cpp, 1.35× over MLX** — are both the single Qwen3-0.6B Q4 cell. The six-model range against llama.cpp is 1.04–1.56×; against MLX 0.90–1.35×. The gain shrinks monotonically with model size and is larger at Q4 than Q8 — exactly what you expect if the mechanism is fixed per-token CPU overhead rather than better bandwidth utilisation: on the 30B MoE, where each step is bandwidth-bound, BaseRT is within 4% of llama.cpp and 1% of MLX.

## Prefill, M4 Pro — Tables 2 and 3

BaseRT absolute prompt-processing throughput (tok/s):

| Model | Quant | pp128 | pp256 | pp512 | pp1024 | pp2048 |
|---|---|---|---|---|---|---|
| Qwen3-0.6B | Q4 | 4,331 | 4,496 | 4,690 | 4,399 | 3,680 |
| | Q8 | 4,288 | 4,434 | 4,623 | 4,354 | 3,648 |
| Llama-3.2-1B | Q4 | 2,490 | 2,577 | 2,615 | 2,577 | 2,453 |
| | Q8 | 2,438 | 2,517 | 2,564 | 2,531 | 2,418 |
| Llama-3.2-3B | Q4 | 888 | 906 | 899 | 888 | 835 |
| | Q8 | 866 | 888 | 893 | 874 | 792 |
| Gemma-4-E2B | Q4 | 1,146 | 1,193 | 1,208 | 1,198 | 1,169 |
| | Q8 | 1,125 | 1,167 | 1,187 | 1,179 | 1,153 |
| Gemma-4-26B-A4B | Q4 | 659 | 704 | 721 | 681 | 626 |
| Qwen3-30B-A3B | Q4 | 738 | 809 | 838 | 810 | 721 |

**BaseRT ÷ llama.cpp** (computed from Table 2; <1.00 means llama.cpp is faster):

| Model | Quant | pp128 | pp256 | pp512 | pp1024 | pp2048 |
|---|---|---|---|---|---|---|
| Qwen3-0.6B | Q4 | 0.90 | 0.90 | 0.94 | 0.95 | 0.90 |
| | Q8 | 0.95 | 0.90 | 0.94 | 0.95 | 0.90 |
| Llama-3.2-1B | Q4 | 0.95 | 0.95 | 0.96 | 0.96 | 0.99 |
| | Q8 | 0.95 | 0.94 | 0.95 | 0.96 | 0.96 |
| Llama-3.2-3B | Q4 | 0.97 | 0.99 | 1.03 | 1.00 | 0.97 |
| | Q8 | 0.95 | 0.96 | 0.96 | 0.98 | 0.94 |
| Gemma-4-E2B | Q4 | 0.93 | 0.93 | 0.93 | 0.94 | 0.97 |
| | Q8 | 1.23 | 1.20 | 1.21 | 1.21 | 1.19 |
| Gemma-4-26B-A4B | Q4 | **1.59** | 1.34 | 1.18 | 1.16 | 1.12 |
| Qwen3-30B-A3B | Q4 | **1.81** | 1.47 | 1.25 | 1.24 | 1.20 |

**BaseRT ÷ MLX** (computed from Table 3):

| Model | Quant | pp128 | pp256 | pp512 | pp1024 | pp2048 |
|---|---|---|---|---|---|---|
| Qwen3-0.6B | Q4 | 1.13 | 1.05 | 1.01 | 0.93 | **0.83** |
| | Q8 | 1.15 | 1.04 | 0.99 | 0.92 | **0.82** |
| Llama-3.2-1B | Q4 | 1.02 | 0.96 | 0.93 | 0.90 | 0.90 |
| | Q8 | 1.04 | 0.96 | 0.93 | 0.89 | 0.87 |
| Llama-3.2-3B | Q4 | 1.01 | 1.01 | 0.98 | 0.94 | 0.89 |
| | Q8 | 1.06 | 0.99 | 0.95 | 0.98 | 0.88 |
| Gemma-4-26B-A4B | Q4 | **1.42** | 1.22 | 1.13 | 1.03 | 0.97 |
| Qwen3-30B-A3B | Q4 | **1.78** | 1.42 | 1.26 | 1.12 | 1.00 |

### Where BaseRT is slower

The abstract's "consistent best-in-class throughput" does not survive its own prefill tables:

- **Dense prefill vs llama.cpp: BaseRT trails on 7 of 8 dense configurations** at every prompt length — by 5–10% on Qwen3-0.6B, 1–6% on Llama-3.2-1B, within ±6% on Llama-3.2-3B, 3–7% on Gemma-4-E2B Q4. The single dense win is **Gemma-4-E2B Q8 (+19–23%)**, which reads as a llama.cpp Q8 Gemma-4 kernel weakness rather than a BaseRT strength (llama.cpp's Q8 is *slower* than its own Q4 there; BaseRT's is not).
- **Dense prefill vs MLX: BaseRT leads only at pp128–256 and loses from pp512 onward**, widening to **0.82–0.90× at pp2048** — i.e. MLX is 10–18% faster on the prompt lengths a coding agent actually sends.
- **Decode vs MLX on Gemma-4-26B-A4B: 0.90×** (62.2 vs 69.3 tok/s). The only decode loss in the paper.
- **MoE prefill vs MLX at pp2048: parity or worse** — 0.97× on Gemma-4-26B-A4B, 1.00× on Qwen3-30B-A3B.
- **vs uzu, prefill: uzu leads on most configurations** by 4–13% on Qwen3-0.6B and 1–3% on Llama, widening with prompt length; BaseRT is ahead only at pp128 on the 8-bit Llama models by 0.3–0.5% (2,438 vs 2,426; 866 vs 863), which is inside the stated ≤10% prefill noise.

The paper's own explanation for the dense-prefill convergence is sound: GEMM-based prefill saturates the matmul units in every runtime, so the throughputs meet — there is no per-token overhead left to remove. That is the [[arithmetic-intensity]] argument, and it means BaseRT's advantage is structurally a **decode** advantage plus a **short-prompt MoE** advantage.

### The MoE prefill gain and why it is larger

Against llama.cpp: **1.59× (Gemma-4-26B-A4B) and 1.81× (Qwen3-30B-A3B) at pp128**, decaying to 1.12× and 1.20× at pp2048. Against MLX: **1.42× and 1.78× at pp128**, decaying to 0.97× and 1.00×. Figure 1's "+78%" is the Qwen3-30B-A3B pp128 cell against MLX.

The paper offers no mechanism beyond noting that *"the margin narrow[s] at longer contexts as the GEMM workload saturates the GPU."* The shape of the curve — largest at the shortest prompt, decaying toward parity — is the decode pattern reappearing: at pp128 a 128-expert MoE routes ~128×k tokens across many experts, so each expert's GEMM has a tiny M and the per-expert dispatch, gather/scatter and synchronisation cost dominates, which is where BaseRT's fused, low-overhead dispatch pays; as the prompt grows each expert GEMM becomes large enough to saturate the GPU and the runtimes converge. **That reading is this wiki's inference, not the paper's claim** — the paper contains no MoE-specific kernel description at all.

## Cross-generation: M3 base decode — Table 4

| Model | Quant | BaseRT | llama.cpp | ratio | MLX | ratio |
|---|---|---|---|---|---|---|
| Qwen3-0.6B | Q4 | 223.0 | 180.5 | 1.24× | 182.3 | 1.22× |
| | Q8 | 124.6 | 99.8 | 1.25× | 113.6 | 1.10× |
| Llama-3.2-1B | Q4 | 125.6 | 106.3 | 1.18× | 112.6 | 1.12× |
| | Q8 | 65.1 | 57.5 | 1.13× | 62.4 | 1.04× |
| Llama-3.2-3B | Q4 | 44.5 | 39.0 | 1.14× | 43.9 | 1.01× |
| | Q8 | 24.2 | 21.2 | 1.14× | 22.2 | 1.09× |
| Gemma-4-E2B | Q4 | 59.0 | 47.1 | 1.25× | — | — |
| | Q8 | 35.4 | 26.5 | 1.34× | — | — |

Dense models only (16 GB rules out the MoEs); **no M3 prefill table is given**. The pattern is the same as M4 Pro with smaller margins: 1.13–1.34× over llama.cpp, 1.01–1.22× over MLX.

## Versus uzu — Tables 5 and 6

uzu (Mirai) is the closest architectural competitor — native Metal, but routing part of the work through MPSGraph. Decode on M4 Pro: BaseRT 1.17–1.19× on Qwen3-0.6B, 1.02–1.06× on Llama-3.2-1B, **0.99×/1.04×** on Llama-3.2-3B (4-bit: 117 vs 119 tok/s). Prefill: uzu leads almost everywhere (see above). The paper's explanation is notable for what it concedes: *"uzu's use of MPSGraph, which can dispatch to the Apple Neural Engine, likely accounts for this prefill advantage: the ANE provides additional compute bandwidth for the GEMM-dominated prefill phase that BaseRT's GPU-only path does not currently exploit."* MetalRT (RunAnywhere) is cited as prior native-Metal work but **not benchmarked**, so the "highest throughput to date" claim excludes it.

## Does the paper compare against MLX on M5?

**No.** Measurements are on M3 base and M4 Pro only. "M5" appears once, as the upper bound of the per-chip kernel configuration table (M1–M5). MLX is v0.31.2 with defaults; on M4 hardware there is no Neural-Accelerator datapath for it to use, so the M4 comparison is fair as far as it goes — but it says nothing about the case that matters for a 2026 buyer. [[exploring-llms-mlx-m5-neural-accelerators-apple-2025|Apple's M5 post]] reports **3.3–4.1× TTFT** over M4 through MLX's automatic use of the GPU tensor units on macOS 26.2+, and its Qwen3-30B-A3B row (3.52×) is the same model where BaseRT's M4 lead over MLX is largest. BaseRT's prefill is hand-written `simdgroup_matrix` GEMM; unless the closed engine has since gained a Metal 4 TensorOps path — nothing in the paper or the September docs says so — the 1.78× pp128 margin would be swamped on an M5. The paper's own ANE concession about uzu is the same shape of problem: a GPU-only kernel library competes on compute-bound prefill against runtimes that can reach extra matrix hardware.

Two third-party M4 findings on [[rigel-metal-tensor-compute-path-m4-max-kumaresan-2026|Rigel]] frame this. First, on an M4 Max the Metal 4.1 `matmul2d` tensor path lowers onto the same `simdgroup_matrix`/FP32-ALU path BaseRT already uses (1.05–1.21× over it), so on M4 there was no matrix datapath for BaseRT to miss — which is consistent with its dense prefill matching the others. Second, Rigel measured a fused online-softmax FlashAttention kernel on M4 as **~9× slower** than a decomposed path that keeps the GEMMs on the matrix instructions; BaseRT reports using exactly such a fused kernel (§3.4) but publishes no attention-kernel microbenchmark, so whether its attention path is a cost or a gain on M4 cannot be read from this paper.

## Feature support for an oMLX-style user

The paper and the repository disagree, because the repository has moved in the ten weeks since submission:

| Feature | Paper (July 2026, §5) | Repo docs, engine 0.2.4 (2026-09-09) |
|---|---|---|
| Continuous batching | **explicitly not implemented** — "does not implement continuous batching, parallel decoding across multiple requests, or tensor parallelism"; listed as future work | `--continuous-batching [N]`, default 8 in-flight lanes, implies `--paged-kv` |
| Paged KV | not mentioned; design pre-allocates a contiguous KV to max context | `--paged-kv` "paged KV cache + block-table dispatch" |
| [[prefix-caching]] | **silent** | `--prefix-cache` (shared across requests), `--prefix-cache-file` persisted across restarts, periodic re-save |
| KV quantization | **silent** | `--kv-bits 4\|8\|16`, default "per-model auto" |
| [[speculative-decoding]] (MTP / DFlash) | **not implemented** — "continuous batching and speculative decoding are natural extensions" | **silent** — no flag, no doc mention |
| Expert / [[weight-streaming]] | **silent**; MoE must fit in RAM (Q8 MoEs excluded from the M4 Pro for that reason) | **silent**; catalog MoE variants are q4-only, "a 30B-class Q4 artifact is around 20 GB" |
| [[speculative-prefill]] / [[sparse-prefill]] | silent | silent |
| CUDA backend | "no CUDA or Vulkan backend exists at present" | NVIDIA GB10 / DGX Spark supported |

> [!warning] The paper measured none of the serving features
> Every table in the paper is single-stream (`tg128`, `pp128–2048`) on the pre-allocated-KV path. The batched, paged, prefix-cached path in the September engine is a different code path with no published numbers; the repo's own benchmarking guide points users at `--continuous-batching` with concurrent clients "rather than the single-stream" benchmark. Nothing here should be read as a throughput claim for that mode, and the paper's decode advantage — fixed per-token CPU overhead — is precisely the cost that batching amortises anyway.

For the user of [[omlx-specprefill-github-2026|oMLX]], the practical delta is: BaseRT now covers continuous batching, prefix caching and KV quantisation, but has **no speculative decoding of any kind, no SpecPrefill, and no expert streaming** — the three levers oMLX users combine on Qwen3.x — and the engine that implements the kernels cannot be read or patched.

## Repository maturity (2026-09-10)

- [basecompute/baseRT](https://github.com/basecompute/baseRT): **211 stars, 16 forks, 23 open issues**; created 2026-06-26 (five days before the arXiv post); last push 2026-09-09.
- **Open-core, not open source.** The repo (Rust CLI, `.base` format spec, C headers, bindings, docs) is Apache-2.0. The engine — `libbaseRT.dylib` with the compiled Metal kernels embedded — ships as a **proprietary prebuilt binary** under its own licence ("you never need to build it from source"). Commits are labelled "sync: open-core source for 0.2.x". Every kernel and dispatch mechanism the paper describes is in the closed half.
- Ten engine releases from v0.1.3 (2026-06-30) to **v0.2.4 (2026-09-09)** — a roughly fortnightly cadence.
- **The README carries no benchmark table.** `benchmarks/results/*.csv` are BaseRT-only runs from an earlier GGUF-input build (pp ≤ 512, no baseline column, no MoE) and differ from the paper: Llama-3.2-1B Q4_0 tg128 **283** vs paper 295; Llama-3.2-3B Q8_0 **63** vs 71; Qwen3-0.6B Q8_0 **290** vs 321. They also cover models the paper omits — Qwen3-4B Q4_K_M 88 tok/s, Qwen3-8B Q8_0 28 tok/s on M4 Pro. The comparison scripts (`three_way_benchmark.sh`, `uzu_benchmark.sh`) are kept "as a reference for methodology rather than a turnkey run."
- Requirements: Apple Silicon M1+ on macOS 14+, or NVIDIA GB10 on Linux/arm64.

## Limitations

- **Vendor benchmark of a closed engine.** Same-machine reproduction is possible only through the prebuilt binary; the kernels cannot be inspected. Treat it as this wiki treats [[exploring-llms-mlx-m5-neural-accelerators-apple-2025|Apple's M5 post]].
- **No ablation.** Five design levers are named; none is isolated. "Framework abstraction is a meaningful source of overhead" is supported only by end-to-end deltas.
- **Noise floor vs claims.** Prefill stddev is "below 10%"; most dense prefill deltas are 1–7%. Decode stddev "below 3%" covers the 1.01×, 1.04× and 0.99× cells.
- **Coverage.** Six models ≤30B, Q4/Q8 only, two chips, no M5, no Q8 MoE, no M3 prefill, no Gemma-4-E2B MLX baseline, MetalRT and vllm-mlx cited but not run.
- **Baseline hygiene is under-specified.** "Default settings" for llama.cpp b9630 — flash-attention and KV-type flags are not stated.
- Everything in the abstract about privacy, latency, connectivity and cost is motivation, not evidence; the Anthropic/OpenAI status-page citations are colour.

Verified: the abstract's 1.56×/1.35× decode figures, Figure 1's +35% and +78%, and every range quoted in §4.2–4.5 reconcile with the tables. Not borne out: "consistent best-in-class throughput from sub-1B to 30B" (dense prefill trails llama.cpp on 7/8 configs and MLX at pp≥1024 on 8/8; one decode loss to MLX; prefill losses to uzu), and Figure 1's "1.15–1.56× over llama.cpp" is the range of its three plotted models, not the six-model 1.04–1.56×.

## Related

- [[mlx]] · [[llama-cpp]] · [[apple]] · [[metal]] — the runtimes and the API.
- [[omlx-specprefill-github-2026]] — the MLX-based server this is most often weighed against; see the feature table above.
- [[exploring-llms-mlx-m5-neural-accelerators-apple-2025]] — the hardware lever this paper does not test.
- [[rigel-metal-tensor-compute-path-m4-max-kumaresan-2026]] — third-party M4 Max measurement of the Metal tensor path BaseRT's GEMMs sit on, and the fused-attention negative result.
- [[quantization-performance]] · [[arithmetic-intensity]] — why the decode gain is real and the dense-prefill gain is not.
- [[methods-to-speed-up-prompt-processing]] — where "swap the runtime" sits among prefill levers: up to 1.8× on short MoE prompts, ≤1.0× on long dense ones.
- [[mixture-of-experts]] · [[kv-cache]] · [[prefix-caching]] · [[speculative-decoding]] · [[chunked-prefill]] · [[weight-streaming]]
- [[hardware-accelerator]] · [[edge-inference]] — the paper's framing of on-device inference as a structural shift.
- [[qwen]] · [[llama]] · [[vllm]] (vllm-mlx is the cited MLX-based competitor)

## References

- Rathnayaka, Waschkowski, Wesemann. *BaseRT: Best-in-Class LLM Inference on Apple Silicon via Native Metal.* [arXiv:2607.00501](https://arxiv.org/abs/2607.00501), 2026-07-01.
- [basecompute/baseRT](https://github.com/basecompute/baseRT) — README, `docs/guides/serving.md`, `docs/reference/engine-releases.md`, `benchmarks/README.md`, `benchmarks/results/*.csv`, GitHub API, read 2026-09-10.
- Cited baselines: llama.cpp (ggml-org, commit b9630), MLX v0.31.2 (ml-explore), uzu (trymirai/uzu), MetalRT (RunAnywhere blog), vllm-mlx ([arXiv:2601.19139](https://arxiv.org/abs/2601.19139)).
