---
title: "Rigel: Reverse-Engineering the Metal 4.1 Tensor Compute Path on the Apple M4 Max GPU"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2606.12765"
authors:
  - Ramchand Kumaresan
year: 2026
venue: null
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Rigel: Reverse-Engineering the Metal 4.1 Tensor Compute Path on the Apple M4 Max GPU

**Source**: [arXiv:2606.12765](https://arxiv.org/abs/2606.12765) (cs.CL, cs.DC), v1 submitted 2026-06-11. Single author, Ramchand Kumaresan, affiliation **Murai Labs**. No venue is stated; this is a preprint. The paper says its harness, Metal kernels, per-cell CSVs and figure scripts are released under the MIT license, but gives no repository URL in the text.

This is the measurement behind a sentence that already appears on [[quantization-performance]], [[methods-to-speed-up-prompt-processing]] and [[exploring-llms-mlx-m5-neural-accelerators-apple-2025]]: on an M4-generation GPU, FP8 matmul is *emulated* at 0.87–0.94× FP16 throughput. It is the "before" picture to the M5 page's "after" — what the GPU tensor path on [[apple|Apple]] Silicon is on a chip with no Neural Accelerators, measured by a third party rather than the vendor.

## What was reverse-engineered

Metal 4.1 (shipping in Xcode 27 / macOS 27.0) adds a tensor compute path: the **Metal Performance Primitives (MPP) `matmul2d`** operation, which contracts two `tensor` operands into a **`cooperative_tensor`** — a register fragment held collectively by the threads of an `execution_simdgroups<N>` scope (N SIMD-groups of 32 threads computing one output tile) and never written to device memory. Operands are either a shader-side `tensor_inline` (a non-owning contiguous view over a device buffer) or a host-bound `tensor_handle` with explicit strides; a `matmul2d_descriptor(m,n,k)` fixes the tile one threadgroup produces, and `dynamic_extent` as `k` makes the op loop over the contraction internally. The Metal Shading Language specification documents this interface, declares the fragment layout "opaque" and "device specific" (§2.22.3.1), and says of each dtype row (fp16, fp8 E4M3/E5M2, fp4 E2M1, block-scaled MXFP4) only that it is *supported* — never whether it is *accelerated*, where the op executes, how wide its accumulator is, or which alignment rules a kernel must satisfy.

The older **`simdgroup_matrix`** instruction (which the 4.1 spec tells developers to abandon in favour of MPP) is not the subject but the yardstick: it is baseline (C) in the GEMM shootout, and the central result is that `matmul2d` lowers onto that same path. The paper frames the community's open question — is Apple's only "tensor core" the `simdgroup_matrix` ALU-utilisation trick, or a dedicated matrix unit like NVIDIA's — and answers it for this chip.

The **discovery bar**: a result counts only if the spec hides or contradicts it. Eleven findings meet it (eight hidden, three contradicted); the OCP-conformant fp8/fp4 representable grids are treated as controls, not contributions.

## Device and method

All measurements are on **one Apple M4 Max — 40 GPU cores, 64 GB unified memory, Mac Studio — under macOS 27.0 beta build 26A5353q** with Xcode 27. The paper is explicit: "Every number is reported 'on M4 Max', we make no claim about other Apple GPUs."

The harness is thin Objective-C++ hosts (allocate, dispatch, time) under a Python layer that owns the checks. Four disciplines gate every number:

- **Checksum gate.** The reference is computed in float64 from the *same quantized bytes* the GPU consumed, so input-quantization error cancels and the residual bounds the kernel's accumulation error alone. Any cell over its per-dtype tolerance is marked garbage and dropped. An independent Python re-verifier re-derived every pass/fail from the committed CSVs and agreed on all fifty cells.
- **Provenance.** Config hash, code commit, input-recipe hash, seed and a full device/OS/toolchain capture are recorded before a run; `make reproduce` redraws every figure from committed CSVs with no GPU.
- **Cheap-baseline gate.** Four pre-registered baselines: (A) the scalar-ALU roofline, (B) a naive threadgroup-tiled GEMM using no tensor primitive, (C) legacy `simdgroup_matrix`, (D) the stock fp16 MPP path. Thresholds fixed in advance: a kernel must beat the strongest baseline by **≥1.10×** to count as a win, and a low-precision format must reach **≥1.5×** its matched fp16 throughput to count as *accelerated* rather than emulated.
- **Reporting.** R=300 repetitions after warm-up; median with a bootstrapped 95% CI (10⁴ resamples, fixed seed). A single-threadgroup "does it run" timing is never reported as throughput.

The CI is not boilerplate. The M4 GPU clock is **bimodal at small sizes**: at 512³ the 300 fp16 samples split into clusters at ~4.65 and ~6.5 TFLOP/s (145 vs 155 reps, CI [5.21, 5.96]); 1024³ runs at ~12.7 with a rare low-clock tail (8/300 reps at ~6.4); 2048³ holds a stable max clock at 14.80 (CI [14.79, 14.80]). A short kernel finishes during the clock ramp and lands in either P-state; a long one sustains the high state.

## Measured throughput

| Problem | fp16 `matmul2d` (TFLOP/s) | fp8 E4M3 ÷ fp16 | fp8 E4M3 (TFLOP/s, derived) |
|---|---|---|---|
| 512³ | 4.65 / 6.5 bimodal, CI [5.21, 5.96] | 0.871× | — (base is bimodal) |
| 1024³ | ~12.7 | 0.927× | ~11.8 |
| 2048³ | **14.80**, CI [14.79, 14.80] | **0.941×** | ~13.9 |

The paper reports fp8 only as a ratio to fp16; the last column is that ratio applied to the fp16 median and is this page's arithmetic, not the paper's. The scalar-ALU fp16 roofline is ~32 TFLOP/s, so the tiled fp16 path peaks at **≈46%** of it and never above.

Other measured figures: MPP beats the naive tiled GEMM by **2.9–5.5×** but `simdgroup_matrix` by only **1.05–1.21×** (ties at small sizes); under sustained `matmul2d` load the GPU rail draws **~42 W** at 100% hardware-active residency, and the IOReport energy rail rises **0.27 → 47.6 W**.

> [!warning] What is *not* measured
> The paper measures throughput for **fp16 and fp8 (E4M3) only**. There is no BF16 row, no INT8 row, and no fp4 throughput — fp4 E2M1 is checked for its representable set and saturation semantics, not speed. It therefore says nothing about whether INT8 has a real datapath on M4. Any "M4 Max INT8/BF16 TFLOPS" figure attributed to Rigel is not in the paper.

## Where `matmul2d` executes: three signals

1. **Throughput ceiling.** 14.8 TFLOP/s at 2048³ is ≈46% of the ~32 TFLOP/s scalar-ALU fp16 roofline and never exceeds it — no hidden datapath beats the ALU ceiling.
2. **Comparison.** The 1.05–1.21× margin over `simdgroup_matrix` is "consistent with MPP lowering onto the simdgroup_matrix and FP32-ALU path rather than onto a separate unit."
3. **Power attribution.** The GPU rail carries the load (~42 W, 100% residency). The Neural Engine half is *indirect*: on this macOS 27.0 beta neither `powermetrics` nor IOReport meters CPU/ANE power (a CPU at 84% residency reads 0 mW) and the ANE compiler rejects the test models, so ANE exclusion rests on signal 2 — an ANE offload would add throughput beyond the GPU-only path, and none appears.

Conclusion: on M4 Max, `matmul2d` runs **entirely on the GPU shader cores, on the same `simdgroup_matrix` and FP32-ALU path any shader uses, with no dedicated matrix datapath**. That is the mechanism behind the fp8 result.

## fp8 is emulated, and what "emulated" means here

Two tiled GEMMs identical in layout, tile and execution scope, differing only in operand dtype (fp16 vs E4M3), both checksum bit-exact. fp8 sustains **0.871 / 0.927 / 0.941× fp16 at 512 / 1024 / 2048**, peaking at 0.94× — nowhere near the pre-registered 1.5× bar. The argument is a one-liner: fp8 operands are one byte against fp16's two, so a memory-bound kernel would be *faster* in fp8; fp8 is *slower*, so the matmul is compute-bound and "fp8 carries pure unpack overhead with no low-precision datapath."

Mechanically, then, "emulated" means the one-byte operands are widened in the shader before the multiply and the multiply itself runs on the same ALU path as fp16 — the fp8 bytes cost conversion work and save no time. Two cautions on precision of that statement: the paper establishes it from throughput and dtype-swapped kernels, not from ISA disassembly, and it does not say whether the unpack target is fp16 or fp32 (it does show the *accumulator* is ≥fp32, below). The contrast the paper draws is Hopper, where fp8 tensor cores roughly double fp16 throughput ([[flash-attention-3-shah-2024|FlashAttention-3]] is the cited exploit of exactly the hardware M4 lacks). The verdict: "On M4 Max, the Metal 4.1 fp8 matmul2d is a memory-footprint and dynamic-range feature, not a throughput feature; the speedup waits for M5."

## Numeric semantics and hidden constraints

- **Accumulator ≥ fp32.** A hostile K=128 contraction — one term of 448 followed by 127 terms of 0.0625, exact sum 455.9375 — would pin at 448 under fp16 accumulation (fp16 spacing near 448 is 0.25, larger than the addends; a 1.74% error). The fp8 `matmul2d` returns 455.9375 bit-exactly.
- **Formats are OCP-exact.** fp8 E4M3, E5M2 and fp4 E2M1 match the OCP specifications across all 256 bit patterns. E5M2 overflows to ±∞; E4M3 has no infinity and clamps to ±448, saturating even a +∞ input.
- **Undocumented 128-byte alignment.** A host-bound `tensor_inline` needs its inner stride 128-byte aligned — contracted dimension a multiple of 128 for fp8, 64 for half, 32 for float — enforced only by a SFINAE note in the headers, so a naive fp8 port of a working half kernel fails with a misleading "no matching constructor".
- **Version gate (contradicted class).** The spec says Metal 4.1 is available from Xcode 26.1; in fact Xcode 26.5 and the 26.6 RC still emit Metal 4.0, only Xcode 27 produces `__METAL_VERSION__` 410, and a 4.1 library refuses to load on macOS 26.5 ("language version 4.1 is not supported on this OS"). Tensors are gated on `__HAVE_TENSOR__` and a deployment target ≥26.2; the 4.1 low-precision formats are absent under the 4.0 toolchain; `int2b` and block-scaled `tensor_blockwise` (MXFP4) are absent despite the feature table.
- **The spec's own advice is overstated.** MPP does beat the deprecated `simdgroup_matrix` at every size, but by 1.05–1.21×: "directionally correct but overstated."

## The opaque fragment layout, recovered

By a pure layout query (instantiate the destination fragment, dump `get_capacity`, `is_valid_element`, `get_multidimensional_index` per lane, no GEMM): for one SIMD-group the partition is an exact tiling with per-lane capacity 2·(M/8)·(N/8), a repeating **8×8 base fragment** in which a lane owns two vertically adjacent rows of one column of its quadrant, replicated across every 8×8 tile. The A and B input fragments share the identical swizzle — the layout is unified across operands and output. This is the Apple-Silicon analogue of the recovered NVIDIA WMMA fragment layout and the lever for layout-aware optimisation.

## Acting on it: epilogue fusion wins, fused attention loses

- `tensor_inline` is contiguous-only and **cannot tile a GEMM**: `slice()` does not offset reads, so only the (0,0) tile computes correctly. A full tiled GEMM is still reachable without the host-bound MTLTensor/MTL4 stack by a contiguous-tile *nt* layout (B pre-transposed, C written tile-contiguously) — that is how the 14.8 TFLOP/s baseline is obtained.
- **Epilogue fusion clears the bar.** Routing a bare GEMM through a `cooperative_tensor` destination matches the host-bound path exactly (no per-contraction round trip to skip). Fusing bias + GELU in-register on the fragment, using the recovered layout, and storing once beats the decomposed GEMM→bias→GELU path by **+12.9% at 1024³ and +6.5% at 2048³**, clearing 1.10× at D ≤ 1024 — "the size range that matters for LLM inference". Each avoided round trip is O(n²) traffic against O(n³) compute, so the win is largest cache-resident and decays with size — the same IO-avoidance principle as [[flash-attention]].
- **Fused attention is a negative result on M4.** A correct scalar FlashAttention (online softmax, no S×S materialisation) is **~9× slower** than a decomposed path that keeps the `simdgroup_matrix` GEMMs, because fusing softmax pushes the matmuls off the matrix path onto scalar code; matrix-unit and split-K variants are correct and faster but still **3.6–5× slower** at S ≤ 8192. On M4 the S×S round trip is cheap on unified memory and the flash kernels are overhead-bound by small tiles and barriers. Cooperative-tensor fusion pays for cheap epilogues that keep the matrix path busy, not for fusions that replace it; a production FlashAttention for Apple Silicon "remains open work."

## What it means for LLM prefill and for formats on M4

Prefill is the compute-bound phase ([[arithmetic-intensity]]), so it takes exactly the throughput the matmul path delivers and nothing from operand bytes. On M4-generation GPUs that means:

- **No precision lever for prefill.** Tier 4 of [[methods-to-speed-up-prompt-processing]] — "lower the compute roof with FP8/INT8" — is an H100-class statement. On M4 the FP8 row is 6–13% *slower* than fp16. The paper calls this "the practical result that most changes how a quantized model should be deployed on M4." It is the same phenomenon as the W4A16 result on [[quantization-performance]] (4-bit weights make prefill 2–12% slower on M1 Pro–M4 Max via llama.cpp Metal): any unpack or dequantise step sits on the critical path of a GEMM that was never waiting on memory.
- **fp8 for weights or KV is a footprint decision, not a speed decision.** The paper's own scope is the matmul: fp8 buys half the bytes and the E4M3 dynamic range at a few percent of matmul throughput. It does not measure decode or any [[kv-cache]] format; whether halved bytes help the bandwidth-bound decode phase is the wiki's inference from [[arithmetic-intensity]], not a Rigel measurement.
- **Attention on M4 should stay decomposed** at the sequence lengths tested; fused flash kernels lose.
- **Fusing cheap epilogues** (bias, activation) on the cooperative-tensor fragment is the one hand-kernel win available, worth 6.5–12.9% at LLM-sized tiles.

## M5 and the Neural Accelerators

The paper is explicit that it measures **a pre–neural-accelerator generation** and nothing else: "The M4 Max sits before the GPU 'Neural Accelerators' Apple reports for the A19/M5 family"; "the expectation going in is a chip that supports the low-precision formats functionally but does not accelerate them; the results bear this out"; "the speedup waits for M5"; "cross-generation claims are out of scope and are the natural M5 follow-up." There is **no M5 measurement, no A19 measurement, and no characterisation of the Neural Accelerators** here. The only M5 numbers held in this wiki remain the vendor's own on [[exploring-llms-mlx-m5-neural-accelerators-apple-2025]] (3.3–4.1× TTFT over M4 at 4,096 tokens), and Rigel's fp16/fp8 figures are the natural M4-side denominator a future third-party M5 study would be measured against.

## Limitations

- One M4 Max, one beta OS build; no cross-chip or cross-generation claim.
- ANE exclusion is indirect (beta OS reports no CPU/ANE power; ANE compiler rejects test models).
- Throughput is reported for fp16 and fp8 E4M3 only — no BF16, INT8, or fp4 throughput.
- Attention kernels are research-grade; a production FlashAttention with register tiling and multiple SIMD-groups per block "may yet win".
- No repository URL appears in the paper text despite the MIT-licence claim.

## Related

- [[exploring-llms-mlx-m5-neural-accelerators-apple-2025]] — the M5 counterpart: what the generation after this one does to the same phase.
- [[above-the-inner-loop-m1-amx-prefill-gemm-bhan-2026]] — the CPU-side sibling measurement: hand-written AMX GEMM on the M1, also finding no low-precision lever (int8 "not a lever" on M1 AMX).
- [[quantization-performance]] · [[methods-to-speed-up-prompt-processing]] — the pages whose FP8/INT8 prefill advice this scopes to hardware with a low-precision datapath.
- [[arithmetic-intensity]] · [[hardware-accelerator]] · [[edge-inference]] · [[quantization]]
- [[flash-attention]] · [[flash-attention-3-shah-2024]] — the fused-attention negative result on M4, and the Hopper fp8 path M4 lacks.
- [[apple]] · [[mlx]] · [[llama-cpp]] — MLX's GPU backend and llama.cpp's Metal backend both sit on the path characterised here.
- [[apple-silicon]] · [[metal]]

## References

- Ramchand Kumaresan, "Rigel: Reverse-Engineering the Metal 4.1 Tensor Compute Path on the Apple M4 Max GPU," [arXiv:2606.12765](https://arxiv.org/abs/2606.12765), June 2026.
- Apple Inc., *Metal Shading Language Specification*, version 4.1 (archived copy dated 2026-06-04) — the document whose hidden and contradicted claims the paper catalogues.
- Z. Jia et al., "Dissecting the NVIDIA Volta GPU Architecture via Microbenchmarking," arXiv:1804.06826 (2018); "Dissecting the NVidia Turing T4 GPU via Microbenchmarking," arXiv:1903.07486 (2019) — the microbenchmarking lineage.
- W. Sun et al., "Dissecting Tensor Cores via Microbenchmarks: Latency, Throughput and Numeric Behaviors," IEEE TPDS 34(1), 2023 (arXiv:2206.02874) — the NVIDIA fragment-layout and accumulator-width recovery this mirrors.
- P. Hübner et al., "Apple vs. Oranges: Evaluating the Apple Silicon M-Series SoCs for HPC Performance and Efficiency," arXiv:2502.05317 (2025); D. Feng et al., "Profiling Apple Silicon Performance for ML Training," arXiv:2501.14925 (2025) — workload-level Apple Silicon studies that do not open the tensor primitive.
- J. Shah et al., "FlashAttention-3," NeurIPS 2024 (arXiv:2407.08608) — Hopper fp8 tensor cores, the hardware support M4 lacks.
- P. Micikevicius et al., "FP8 Formats for Deep Learning," arXiv:2209.05433 (2022); OCP, *Microscaling Formats (MX) Specification* v1.0 (2023) — the representable sets M4 implements exactly.

## Related 2026 sources

- [[basert-native-metal-inference-apple-silicon-rathnayaka-2026]] — a runtime built on the same `simdgroup_matrix` path this paper characterizes; loses dense prefill to MLX from 512 tokens up
