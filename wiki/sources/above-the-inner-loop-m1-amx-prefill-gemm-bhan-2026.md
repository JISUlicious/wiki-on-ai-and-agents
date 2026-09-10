---
title: "Above the Inner Loop: Exceeding Accelerate at LLM Prefill GEMM on the M1 AMX"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2606.25426"
authors:
  - Deyvik Bhan
year: 2026
venue: null
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Above the Inner Loop: Exceeding Accelerate at LLM Prefill GEMM on the M1 AMX

**Source**: [arXiv:2606.25426](https://arxiv.org/abs/2606.25426) (cs.PF; ACM classes C.1.2, C.4), v1 submitted 2026-06-24, "11 pages, 2 figures, 6 tables". **Single author**, Deyvik Bhan; the full text gives the affiliation as **Georgia Institute of Technology** (the arXiv abstract page lists none). No venue is stated; this is a preprint. Code, benchmarks, the prior-art map (`docs/LITERATURE.md`) and the investigation log are at [github.com/dbhan08/inferc](https://github.com/dbhan08/inferc).

> [!note] This is the CPU path, not the path a Mac normally runs LLMs on
> On Apple Silicon the GPU is what [[mlx]] and [[llama-cpp]]'s Metal backend use, and every Apple Silicon prefill/decode table in this wiki ([[quantization-performance]], [[methods-to-speed-up-prompt-processing]], [[exploring-llms-mlx-m5-neural-accelerators-apple-2025]]) is a GPU number. This paper is about the *other* matrix engine: the **Apple Matrix coprocessor (AMX)** attached to the CPU clusters on M1–M3, which Apple's Accelerate framework uses for fp32 SGEMM and which llama.cpp reaches only through its `ggml-blas` backend with Metal off. **The paper never benchmarks the GPU or Metal** — "GPU" appears only in its related-work section and "Metal" only inside a cited title — so nothing here says how the AMX kernel compares to the Metal path on the same machine. Read it as a micro-architecture result about where a CPU-side GEMM's headroom comes from, not as a way to make a Mac faster than its GPU already is.

## Problem

During prefill on an Apple Silicon CPU, fp32 GEMMs dominate per-layer work, and Accelerate is the standard provider: it dispatches SGEMM to a matrix coprocessor — AMX on M1–M3, Arm SME on M4 and later — through a single shape-agnostic entry point tuned to be reasonable everywhere rather than optimal anywhere. MpGEMM (Deng et al., arXiv:2512.21473) had shown a hand-written SME kernel beating Accelerate by 1.23× on an M4 Pro. The question here is the analogous one on the M1 — "the older but far more widely deployed" part, which with M2 and M3 "constitutes the majority of deployed Apple Silicon" — and specifically *where* any headroom over Accelerate comes from. The answer is structural: not from a faster inner loop, which is at the hardware limit, but from two deployment-level levers Accelerate forgoes at these shapes.

## Hardware and the AMX inner loop

**M1 (T8103)**: four P-cores at 3.228 GHz nominal, four E-cores; 128 KB L1d per core, 12 MB L2 shared across the P-cluster, no L3. One AMX block per cluster — one shared by the four P-cores, a second lower-throughput block shared by the four E-cores. AMX is not in Apple's public ISA; instructions are `.word` directives after a reserved Arm64 opcode, using the reverse-engineered corsix/amx and dougallj encodings (the vendored header matches corsix HEAD, verified against the corsix emulator). The fp32 primitives: `AMX_LDX` (64 or 128 bytes into one or two X registers), `AMX_LDY` (64 bytes into Y), `AMX_FMA32` (a 16×16 outer-product FMA, `Z[j][i] += X[i]·Y[j]`, four Z banks), `AMX_STZ`/`AMX_LDZ` (store/load one Z row). macOS Tahoe 26.5.1, Apple clang 21.0.0 at `-O3`.

**The load-issue bound (Table 2).** A cache-resident microbenchmark issuing only FMA32 outer products reaches **~1,525 GFLOPS on one thread** (900–1,600 across thermal states). Interleave even one operand load with the FMA32 stream and single-thread throughput collapses into a **610–680 GFLOPS band** and stays there:

| Inner loop (all L1-resident) | loads : FMA32 | GFLOPS |
|---|---|---|
| FMA32 only, no loads | 0 : 4 | ~1,525 |
| + 1 paired load | 1 : 4 | 677 |
| + 1 paired load, no data dependency | 1 : 4 | 679 |
| + 2 paired loads (32×32 tile) | 2 : 4 | 609 |
| + 3 loads (16×64 tile, deployed) | 3 : 4 | 643 |
| 4-iteration phase batch | 8 : 16 | ~610 |

It moves little with load count, load pairing, dependency, output-tile shape, or batching loads ahead — consistent with loads and FMA32 contending for a single AMX issue slot, with the 1,024-element Z accumulator capping the work at four FMA32 per load group. No single-thread rearrangement escapes it. The `LDX_pair` encoding (halving load count) and software `PRFM` prefetch change nothing.

**Two blocks across threads (Table 5).** The single-thread floor is not the multi-thread ceiling: one thread's loads issue while another's FMA32 occupy the block.

| Threads | loaded (GFLOPS) | pure-FMA (GFLOPS) |
|---|---|---|
| 1 | 673 | 1,528 |
| 2 | 1,219 | 1,419 |
| 3 | 1,373 | 1,418 |
| 4 | 1,324 | 1,332 |
| 6 | 1,435 | 1,542 |
| 8 | **1,483** | 1,581 |

The pure-FMA aggregate barely moves (+3% net, one thread already saturates the issue port); the loaded aggregate rises 2.2× because loads slot into the gaps. Threads beyond four spill onto the E-cluster's second block.

## Method: the two levers

The kernel is Goto–BLIS three-level blocking on a non-standard instruction set: A is pre-transposed once to row-major `At[K, M]` so a 16-element column slice loads into one Y register; B is packed into contiguous `packB[Kc, Nc]` panels; per iteration the microkernel reads four X registers of B (two `LDX_pair`) and one Y of A and issues four FMA32 across the four Z banks (4-way ILP). Correctness hinges on the Z carry — for `pc > 0` the partial accumulator is reloaded from C with LDZ and stored with STZ every iteration, skip-Z asserted only at `pc = 0, kk = 0`; without that discipline Z state leaks between tiles and drifts silently.

1. **Fine multi-thread panels that fill the second AMX block.** Work is parallelised over column panels of width `Nc` (⌈N/Nc⌉ of them, dispatched through Grand Central Dispatch, no pinning). A coarse `Nc = 512` at the QKV shape gives four panels — enough for the P-cluster, not the E-cluster — and stalls near **630 GFLOPS**; `Nc = 64` gives 32 panels, fills all eight cores and both blocks, and lifts the same shape to **~1,200**. Across the QKV and FFN-down shapes of all three scales, 512→64 is worth **85–102% (geometric mean ≈1.9×)**; the gain persists even at N = 4,096 where eight 512-wide panels already cover eight cores, because a 512-wide packed panel also carries an eightfold larger shared-L2 footprint. Mis-tuning this one value "costs nearly 2×."
2. **Pre-packing the constant weight at load.** In LLM inference B is a weight, constant across every token and step of a session, but `cblas_sgemm` and `BNNSMatMul` receive it per call and re-pack it every time. The kernel packs each weight once into `[K, Nc]` panels at model load (~480 MB at the LM-head shape, paid once) and runs only the LDY/LDX/FMA32/LDZ/STZ loop per call. This also unlocks `Kc = 2,048`, which halves the Z-reload passes and sizes the per-tile A slice to exactly 2,048 × 16 × 4 = 128 KB, the P-core L1 — a depth a per-call kernel cannot afford to pack.

The deployed pair `(Nc, Kc) = (64, 2,048)` comes from one offline sweep (`Nc ∈ {64…512}`, `Kc ∈ {256…2,048}`, rejecting anything not bit-identical to Accelerate) and is used uniformly at all twelve shapes.

**Negative results (Table 1).** Operand streaming with no B pack: 0.15–0.5× of packed (cache-hostile strided B). Microkernel restructuring (tile, pairing, pipelining, batching): no change. Explicit thread-count pinning: within noise of GCD at `Nc = 64`. 2-D output tiling (splitting M into 16- or 32-row blocks): lowers throughput — "AMX wants the full M = 128 per work item."

## Results: twelve prefill GEMMs at S = 128 (Table 3)

Every row is M = S = 128; shapes are (N, K); GFLOPS, median over eleven isolated binary invocations (≥15 timed trials each after warm-up), all backends at their default threading in one process per invocation. Every proposed-kernel cell is bit-identical to Accelerate (max-abs-diff 0 over a coprime-stride sample of C). "GPT-2-style" is an H = 2,048, FFN×4, V = 60,000 configuration used to populate a large-vocabulary LM-head case — not [[gpt-2|GPT-2]]-small (H = 768).

| Model | Op | (N, K) | cblas_sgemm | BNNSMatMul | **proposed** | ÷ BNNS |
|---|---|---|---|---|---|---|
| GPT-2-style | QKV | (2048, 2048) | 614 | 809 | **1221** | 1.51 |
| GPT-2-style | FFN1 (up) | (8192, 2048) | 403 | 681 | **1239** | 1.82 |
| GPT-2-style | FFN2 (down) | (2048, 8192) | 527 | 653 | **1144** | 1.75 |
| GPT-2-style | LM head | (60000, 2048) | 831 | 838 | **1211** | 1.45 |
| TinyLlama-1.1B | QKV | (2048, 2048) | 614 | 800 | **1214** | 1.52 |
| TinyLlama-1.1B | FFN1 | (5632, 2048) | 839 | 795 | **1230** | 1.55 |
| TinyLlama-1.1B | FFN2 | (2048, 5632) | 520 | 680 | **1173** | 1.73 |
| TinyLlama-1.1B | LM head | (32000, 2048) | 899 | 850 | **1252** | 1.47 |
| Llama-7B | QKV | (4096, 4096) | 369 | 725 | **1236** | 1.70 |
| Llama-7B | FFN1 | (11008, 4096) | 804 | 830 | **1246** | 1.50 |
| Llama-7B | FFN2 | (4096, 11008) | 336 | 697 | **1125** | 1.61 |
| Llama-7B | LM head | (32000, 4096) | 735 | 843 | **1216** | 1.44 |

- **Geometric mean 1.58× over BNNSMatMul** (range 1.44–1.82) and **2.04× over cblas_sgemm**, all twelve shapes. The kernel sits at a nearly shape-independent **1,125–1,252 GFLOPS**, so its margin is the inverse of the baseline's own throughput: largest at FFN-down (K > N; BNNSMatMul at 653–697; geomean 1.70×), smallest at the LM head (N ≫ K; 838–850; 1.45×).
- Coefficient of variation ≤3.6% at every shape for the kernel, against 12% for cblas at small QKV and 5.1% for BNNSMatMul.
- BNNS-FC (the fully-connected filter, which does pre-pack at construction) collapses to 467–478 GFLOPS at the large-N Llama shapes; measured only there, as context.
- For scale: OpenBLAS 0.3.33 NEON sgemm sustains ~200–300 GFLOPS on an M1 P-core, so every AMX backend is in a regime NEON cannot reach.

**Against BNNS Graph (Table 4)** — macOS 15+'s ahead-of-time graph compiler, which repacks constant weights at compile time (untimed, matching the kernel's amortised pre-pack; fp32 storage and compute forced): the kernel leads at all twelve, **geometric mean 1.17× (range 1.09–1.41)**. BNNS Graph is bit-exact against cblas only at the three square QKV shapes (where the margin is a clean bit-exact-vs-bit-exact **1.09×**, with non-overlapping per-invocation distributions); at all nine rectangular shapes it differs by 2.4e-4 to **1.4e-3** — about a hundred times the fp32 rounding floor, identical to the last digit across invocations — so it is computing them at reduced precision despite fp32 storage.

**Ablation (§4.4).** The multi-threaded kernel *without* pre-packing (re-tuned to `Kc = 1,024`) beats BNNSMatMul at seven of twelve (geomean 1.07): all six K ≥ N shapes win by 1.05–1.46 on panel granularity alone; five of six N > K shapes lose at 0.87–0.92 because packing the large-N weight per call dominates. Pre-packing converts those, and unlocks the deeper `Kc`. A batch-size sweep of the per-call single-thread kernel at QKV over S ∈ {16…512} never beats single-thread cblas (0.54–0.70×) — "the headline advantage is a property of the pre-packed deployment, not of the inner kernel at any batch size."

**Occupancy probe (§4.6).** AMX has no public performance counter on M1, but each block can be driven only by a core in its cluster, so per-core active residency (`host_processor_info`, baseline-subtracted, as core-equivalents of four) is a sound proxy. At QKV: `cblas_sgemm` and `BNNSMatMul` hold the **E-cluster at 0.00** and keep 0.4–1.3 P-core-equivalents busy; the kernel at `Nc = 64` drives **3.0–3.5 on the P-cluster and 1.9–2.5 on the E-cluster**. Same at FFN1, across six runs — the idle second block is observed, not inferred.

## End-to-end prefill GEMM and llama.cpp

**Standalone harness (Table 6)** — the full prefill GEMM sequence in layer order at S = 128, fp32 (Q, K, V, attention-output, FFN-up, FFN-down per block, then one LM head; SDPA omitted), with the activation transpose *inside* the timed region for the kernel:

| Model | H | F | V | L | cblas (ms) | BNNS (ms) | **ours (ms)** | vs BNNS | vs cblas |
|---|---|---|---|---|---|---|---|---|---|
| TinyLlama-1.1B | 2,048 | 5,632 | 32,000 | 22 | 432 | 341 | **240** | 1.42× | 1.80× |
| Llama-7B | 4,096 | 11,008 | 32,000 | 32 | 3,207 | 1,799 | **1,200** | 1.50× | 2.67× |

These sit just below a FLOP-weighted composition of Table 3 (1.58× and 1.62× over BNNSMatMul) because the transpose is now paid every call.

**Inside [[llama-cpp]] (§4.7).** llama.cpp's `ggml-blas` backend routes every prefill weight matmul through one `cblas_sgemm(NoTrans, Trans)` call whose `src0` is the constant fp32 weight `W[N, K]`. The shim intercepts exactly that call for the constant-weight fp32 case, pre-packs each weight at first use (107 matrices), dispatches the `Nc = 64, Kc = 2,048` kernel, and falls back to `cblas_sgemm` for everything else (attention scores, small shapes); an environment variable toggles it so one binary runs both arms on the same file.

- **Model and format: TinyLlama-1.1B, fp32 [[gguf|GGUF]] (4.1 GiB).** fp32 only — "the quantized ggml path, where dequantization is fused into the matmul, is a different code path our kernel does not target."
- **`llama-bench`, four threads, 128-token prefill: 291 → 420 tokens/s, 1.44× full-forward**, median over twenty iterations with the cold first excluded (it carries the one-time packing and GCD pool spin-up); CoV 4.7% baseline, 5.3% ours. A first-call recomputation through `cblas_sgemm` confirms the output is **bit-identical** (max abs diff 0).
- GEMM dominates this fp32 prefill: baseline ~440 ms per 128-token forward against the standalone 432 ms cblas GEMM time. The 1.44× sits below the 1.80× GEMM-only ratio not because non-GEMM work dilutes it but because the in-engine kernel runs ~57 ms slower than in isolation (305 ms measured vs ~248 ms / ~516 tok/s / 1.77× if it had matched) — GCD threads contending with llama.cpp's own pool, and the activation transpose paid per call rather than shared across the Q/K/V trio.
- **The gain is confined to short prefill: 1.10× at 256 tokens, and it *reverses* to 0.82× at 512**, where the per-call activation transpose (the weights are pre-packed, the activations are not, and the transpose grows with M) and Accelerate's stronger large-M scaling overtake it.

## How this relates to the GPU path, and to the M4-side result

Nothing in the paper places these numbers against Metal. The kernel's ~1.2 TFLOPS fp32 aggregate is a CPU-cluster figure on a 2020 part; the wiki's Apple Silicon prefill tables ([[quantization-performance]]) are llama.cpp *Metal* numbers on M1 Pro through M4 Max, and MLX runs on the GPU. A reader wanting "how much faster is the Mac at prefill" should look there; this paper answers "where is the headroom in the fp32 CPU BLAS path that llama.cpp's Accelerate build uses when Metal is not in play." Three details do transfer:

- **No low-precision lever on M1 AMX either.** The limitations section states the int8 path "is not a lever: the M1 vecint primitive runs at about five cycles per instruction," and that the AMX fp16 path (~2× the fp32 limit) is under-exploited by Accelerate's BNNS fp16 routines — left as future work. That is the CPU-side rhyme with [[rigel-metal-tensor-compute-path-m4-max-kumaresan-2026|Rigel]]'s finding that the M4 Max GPU has no fp8 datapath: on pre-M5 Apple Silicon, neither matrix engine rewards a narrower format with throughput.
- **The prefill/decode split is the same.** The result is for S = 128, compute-bound; decode's M = 1 GEMV is bandwidth-bound and out of scope — the [[arithmetic-intensity]] argument again.
- **Weight pre-packing is a deployment fact, not a kernel fact.** For a single isolated GEMM the kernel is no faster than BNNSMatMul; the win exists only because a serving session reuses the packed weight across every call. The same amortisation logic underlies [[prefix-caching]] and quantised-weight layouts in [[gguf]].

## Limitations (the paper's own)

- **fp32 only.** BNNS Graph is faster when allowed its default reduced precision; that headroom is unaddressed. The 1.09× where both hold fp32 is the conservative comparison.
- **Requires the pre-packed deployment**; a stateless GEMM interface gets none of it.
- **llama.cpp gain is confined to the fp32 BLAS path** and to ≤128-token prefill; the quantised ggml path, a hoisted activation transpose, and longer prefill are the deferred validation.
- **Prefill only**, not decode.
- **Generalisation splits.** The load-issue bound is a property of the AMX FMA32/load issue port and is expected to hold on M1–M3; the ~1,480 GFLOPS two-block aggregate and the `Nc = 64` that reaches it depend on the 4P + 4E single-P-cluster layout — shared by base M2 and M3, not by M2 Pro/Max/Ultra (multi-cluster), and replaced entirely by SME on M4 and later.
- One M1 unit; no control for silicon-to-silicon variation (ratios are formed within a process, so thermal excursions largely cancel).

## Related

- [[rigel-metal-tensor-compute-path-m4-max-kumaresan-2026]] — the GPU-side sibling: Metal 4.1 `matmul2d` on M4 Max, fp8 emulated at 0.87–0.94× fp16.
- [[exploring-llms-mlx-m5-neural-accelerators-apple-2025]] — the GPU generation step that actually moves Mac prefill (3.3–4.1× TTFT); the accelerator this CPU path does not have.
- [[quantization-performance]] · [[methods-to-speed-up-prompt-processing]] — the Metal-backend Apple Silicon numbers this should not be confused with.
- [[llama-cpp]] · [[gguf]] · [[llama]] — the engine, the file format and the model family used for the end-to-end run.
- [[arithmetic-intensity]] · [[hardware-accelerator]] · [[edge-inference]] · [[apple]] · [[mlx]]
- [[amx]] · [[apple-silicon]] · [[metal]] — no pages yet (unresolved).

## References

- Deyvik Bhan, "Above the Inner Loop: Exceeding Accelerate at LLM Prefill GEMM on the M1 AMX," [arXiv:2606.25426](https://arxiv.org/abs/2606.25426), June 2026. Code: [github.com/dbhan08/inferc](https://github.com/dbhan08/inferc).
- C. Deng, W. Yang, J. Fang, D. Dong, "Demystifying ARM SME to Optimize General Matrix Multiplications" (MpGEMM), arXiv:2512.21473, 2025 — the SME/M4 Pro counterpart, 1.23× over Accelerate.
- J. Zhou, "Performance Analysis of the Apple AMX Matrix Accelerator," M.Eng. thesis, MIT, 2025 — in-place masked-outer-product GEMM exceeding Accelerate on M2.
- P. Hübner, A. Hu, I. Peng, S. Markidis, "Apple vs. Oranges: Evaluating the Apple Silicon M-Series SoCs for HPC Performance and Efficiency," arXiv:2502.05317, 2025.
- B. Kilictas, F. Alpay, "Bare-Metal Tensor Virtualization: Overcoming the Memory Wall in Edge-AI Inference on ARM64," arXiv:2601.03324, 2026 — a NEON engine that trails the Accelerate/AMX baseline by 4.87×.
- P. Cawley, [corsix/amx](https://github.com/corsix/amx); D. Johnson, [dougallj](https://github.com/dougallj) — the reverse-engineered AMX encodings; P. Turner, [amx-benchmarks](https://github.com/philipturner/amx-benchmarks).
- K. Goto, R. A. van de Geijn, "Anatomy of High-Performance Matrix Multiplication," ACM TOMS 34(3), 2008 — the blocking structure.
- A. Benazir, F. X. Lin, "Benchmarking and Characterization of Large Language Model Inference on Apple Silicon," POMACS 9(3), 2025 — the GPU/unified-memory-side characterisation.
- S. Catalán et al., "A comparative performance and efficiency analysis of Apple's M architectures: A GEMM case study," FGCS 180, 2026 — CPU, GPU, AMX and Neural Engine GEMM across M1–M4.
- Apple, "What's new in BNNS Graph," WWDC25 session 276 — the automatic weight repacking BNNS Graph performs.
