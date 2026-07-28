---
title: "FlashKDA: high-performance Kimi Delta Attention kernels"
type: source
created: 2026-07-28
updated: 2026-07-28
sources:
  - flash-kda-moonshot-2026.md
year: 2026
venue: "Open-source release (GitHub, MIT)"
authors:
  - Moonshot AI (Kimi Team)
tags:
  - 2026
status: complete
importance: medium
---

# FlashKDA: high-performance Kimi Delta Attention kernels

**[[moonshot-ai|Moonshot AI]] (Kimi Team)** — [github.com/MoonshotAI/FlashKDA](https://github.com/MoonshotAI/FlashKDA), MIT licensed. Deep-dive doc dated **2026-04-20**; benchmark file generated **2026-04-22**.

Not a paper — a **CUTLASS kernel library**. It is the optimized implementation of **Kimi Delta Attention (KDA)**, the [[linear-attention]] mechanism introduced in [[kimi-linear-kimi-team-2025|Kimi Linear]] and used in Kimi K3 alongside [[attention-residuals-kimi-team-2026|AttnRes]].

## Why it matters here

It is a concrete demonstration that **prefill speed is a software problem before it is a hardware problem**: ~2× on unchanged H20 silicon, from kernel engineering alone, with correctness tested as exact match against a torch reference.

Adoption cost is near zero — it is a **drop-in backend for `flash-linear-attention`**, auto-dispatched from `fla.ops.kda.chunk_kda` once installed (`flash-linear-attention >= 0.5.0`), with `FLA_FLASH_KDA=0` to fall back to the Triton path. Requires **SM90+ (Hopper)**, CUDA 12.9+, PyTorch 2.4+.

## Benchmarks (H20, `BENCHMARK_H20.md`)

`warmup=30, iters=200, repeats=5`. **T=8192, H=96, D=128**:

| Case | `flash_kda` | `fla_chunk_kda` | **Speedup** | vs gated delta rule |
|---|---:|---:|---:|---:|
| Fixed | 2.6220 ms | 4.8388 ms | **1.85×** | 1.22× |
| Varlen `[1300, 547, 2048, 963, 271, 3063]` | 2.3449 ms | 4.8291 ms | **2.06×** | 1.30× |
| Varlen `1024 × 8` | 2.0432 ms | 4.6723 ms | **2.29×** | 1.43× |

At **H=64, D=128**: 1.95× / 1.91× / **2.31×** on the same three cases.

**The win grows with variable-length batching** (1.85× fixed → 2.29× varlen), which is the practically relevant direction since production batches are ragged rather than padded.

> [!note] Two precision caveats
> - Moonshot's announcement quotes **"1.72×–2.22× prefill speedup"**, while the repo's own `BENCHMARK_H20.md` shows **1.85×–2.31×** vs `chunk_kda`. Prefer the benchmark file — it is the primary artifact and states its configuration.
> - These are **forward-kernel** benchmarks ("KDA forward benchmark"), not end-to-end model prefill. As with [[flash-attention]], a kernel speedup is an upper bound on the end-to-end effect, not the same number.

## Design decisions (from the v1 deep-dive)

**`CHUNK = 16`, against Flash Linear Attention's `CHUNK = 64`** — three reasons, and they interlock:
1. **bf16 numerical range.** With the gate `lower_bound = -5`, chunk 16 keeps `exp(cumsum(g))` inside bf16's representable range, *eliminating the intra-chunk rescaling tricks larger chunks require*.
2. **Cheap matrix inversion.** A 16×16 inverse is dramatically cheaper than 64×64, and is computable directly from a **Neumann-series expansion** with no further decomposition.
3. **SM80-only MMA path.** All chunk-16 math maps onto SM80 MMA instructions, keeping the kernel portable across modern NVIDIA GPUs rather than tied to one architecture.

**Two kernels, not one fused kernel.** Split along natural parallelism axes:
- **K1** (token-parallel, grid `N × H × num_chunks`): gate activation → L2 norm → decay → `L`/`Mqk` construction → matrix inversion
- **K2** (head-parallel only, grid `N × H`): chunk-by-chunk delta-rule recurrence → output projection → state accumulation

Early single-fused-kernel prototypes had K1's token-parallel work **bottlenecked by K2's much lower recurrence parallelism, leaving a large fraction of SMs idle**. Splitting gave **≥15% end-to-end speedup** and made each stage independently tunable. This is a nice general lesson: fusion is not free when the fused stages have mismatched parallelism.

**Precision choices:**
- Recurrent state stored **on-chip in bf16** — halves shared-memory footprint and removes an fp32→bf16 cast from the critical path of every bf16 GEMM feeding the state. Validated as no measurable accuracy loss *provided the state update itself uses fp32 FMA*.
- Sigmoid via PTX **`tanh.approx.f32`**.
- The 16×16 inverse computed in **fp16, not bf16** — the inverse's elements are bounded in `[-1, 1]`, so fp16's narrower dynamic range suffices, while its extra mantissa gives the Neumann expansion more headroom and avoids a cast.

**Other optimizations:** base-2 exponent (`ex2.approx.ftz.f32`, removing the change-of-base FMA); aggressive shared-memory reuse plus `__launch_bounds__(256, 8)` to trade minor register spilling for far more blocks per SM; and `MOVM_T` register-file transposes in K2 that eliminate every intermediate shared-memory round trip between stages.

## Concepts & entities

- [[linear-attention]] — the family KDA belongs to; this is its production kernel.
- [[kimi-linear-kimi-team-2025]] — the architecture paper introducing KDA.
- [[moonshot-ai]] — publisher; [[attention-residuals-kimi-team-2026]] — the sibling Kimi Team line, combined with KDA in Kimi K3.
- [[flash-attention]] — the analogous kernel-engineering effort for *exact quadratic* attention.

## References

- [github.com/MoonshotAI/FlashKDA](https://github.com/MoonshotAI/FlashKDA) — README, `BENCHMARK_H20.md`, `docs/20260420-flashkda-v1-deep-dive.md`
- _Original source: `sources/flash-kda-moonshot-2026.md` (README + benchmark + deep-dive, concatenated)_
