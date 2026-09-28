---
title: "A Persistent-State Dataflow Accelerator for Memory-Bound Linear Attention Decode on FPGA"
type: source
created: 2026-09-09
updated: 2026-09-09
sources: []
arxiv_id: "2603.05931"
authors:
  - Neelesh Gupta
  - Peter Wang
  - Rajgopal Kannan
  - Viktor K. Prasanna
year: 2026
venue: null
status: complete
importance: high
tags:
  - 2026
  - paper
---

# A Persistent-State Dataflow Accelerator for Memory-Bound Linear Attention Decode on FPGA

**Neelesh Gupta, Peter Wang, Rajgopal Kannan, Viktor K. Prasanna** (University of Southern California; Kannan at DEVCOM Army Research Office) — [arXiv:2603.05931](https://arxiv.org/abs/2603.05931), cs.AR / cs.LG, submitted 6 Mar 2026, v1, 6 pages / 6 figures. **No venue stated.** NSF-funded (CNS-2009057, CSSI-2311870, CCF-1919289, SPX-2333009).

> [!important] What this accelerator actually contains
> It holds **only the recurrent state**, for a **single Gated DeltaNet layer**. There are no model weights on the board — no Q/K/V projections, no output projection, no FFN/MoE, no softmax-attention layers. The host computes q, k, v and the gate inputs and streams ~48.5 KB of them across AXI *per token, per kernel invocation*. This is a **coprocessor for one operation**, not a drop-in inference engine. Everything below should be read with that scope in mind; the paper's own future work confirms it ("extend the persistent-state datapath to support prefill … and co-acceleration of the remaining softmax layers").

## Problem

This is the decode-side mirror of [[fast-prefill-fpga-sparse-attention-jayanth-2026]], and it lands directly on the architecture family this wiki already tracks closely. [[linear-attention|Gated DeltaNet]] replaces the growing [[kv-cache]] with a fixed-size recurrent state matrix S ∈ ℝ^(d×d). The paper's framing claim — **"Hybrid LLMs like Qwen3-Next use 75% GDN layers"** — checks out and matches this wiki independently: [[qwen3.8-flash-next]] is 48 layers laid out as 12 × (3 × Gated DeltaNet → 1 [[sparse-attention|sparse attention]]), i.e. **36 of 48 = 75%**, the same 3:1 ratio [[kimi-linear-kimi-team-2025|Kimi Linear]] reached separately. So GDN decode is the *dominant per-token primitive* in this model class, and accelerating it is not a niche exercise.

The sharp observation is about **[[arithmetic-intensity]]**, and it inverts the usual reading of linear attention. Fixed-size state is a memory-*capacity* win but an [[arithmetic-intensity|intensity]] loss: every layer must read and write its **entire** state matrix every single token, no matter how short the sequence. At batch 1 that gives:

| Architecture (batch-1 decode) | Arithmetic intensity |
|---|---|
| MHSA Transformer with GQA | **~1 FLOP/B** |
| Gated DeltaNet | **~0.87 FLOP/B** |
| DeltaNet, Mamba, Mamba-2 | also **< 1 FLOP/B** |
| H100 FP32 ridge point | **25.6 FLOP/B** |
| *This design* | **~88 FLOP/B** |

So **subquadratic models are *more* memory-bound at decode than the Transformers they replace**, not less — the paper calls this architectural, not algorithmic, and notes that better kernels (FlashInfer is cited) reduce software overhead but cannot remove the HBM round trip. That is a genuinely useful correction to the intuitive story around [[linear-attention]] and worth carrying back to [[llm-inference-roofline-yuan-2024]].

The enabling insight is a size coincidence: for a Qwen3-Next-style layer with h_v = 32 value heads and d = 128 in FP32, the whole state is **32 × 128 × 128 × 4 B = 2 MB**, against the Alveo U55C's **17.6 MB of BRAM**. BRAM is a software-managed scratchpad that persists across kernel invocations; the H100's 50 MB L2 is a hardware-managed cache with no persistence guarantee, so the GPU must round-trip 4 MB through HBM every token. Pin the state on-chip and state I/O goes to **exactly zero**.

## Mechanism

Four composed techniques, targeting Qwen3-Next's layer shape (h_q = h_k = 16, h_v = 32, d = 128, FP32 throughout — **no quantization anywhere**, unlike paper 1):

1. **Persistent on-chip state.** 32 state matrices in static HLS arrays mapped to dual-port BRAM, retained across all token steps. Per-token off-chip traffic falls from ~4.24 MB to ~48.5 KB.
2. **Fused five-phase pipeline.** The naive recurrence needs *three* passes over S (retrieval, update, output). An algebraic restructuring — Sₜᵀq = g·Sₜ₋₁ᵀq + (qᵀk)Δv — lets the partial output be computed in the *same read pass* as the retrieval, so the updated state is never re-read. **Three passes → one read + one write**, T_iter from ~3,072 to ~2,106 cycles (**1.46×**).
3. **GVA-aware paired-head parallelism.** Qwen3-Next uses 2:1 Grouped Value Attention — each q/k pair serves two value heads with independent states. Broadcasting q and k to both doubles compute per load without duplicating storage.
4. **Dataflow pipelining.** Prepare(n+1) ∥ Compute(n) ∥ Store(n−1); the state array is indexed [iter][h][i][j] so the HLS compiler can statically prove disjointness.

Design knob is H_iter, value-heads per iteration, swept over {2, 4, 8, 16}.

## Measured results

| | NVIDIA H100 PCIe | AMD Alveo U55C |
|---|---|---|
| Technology | TSMC 4N | TSMC 16 nm |
| FP32 peak | 51 TFLOP/s | ~2.7 TFLOP/s |
| Memory BW | 2.0 TB/s (HBM3) | 460 GB/s (HBM2e) |
| On-chip memory | 50 MB L2 **cache** | 17.6 MB BRAM **scratchpad** |
| Board TDP | **350 W** | **150 W** |

Per-token latency for one GDN layer, batch 1:

| Config | Cycles | @300 MHz | @250 MHz | Speedup vs. GPU | Resource (BRAM/DSP/FF/LUT) |
|---|---|---|---|---|---|
| H100 (PyTorch NVLabs reference) | — | **285 μs** | — | 1.0× | — |
| H_iter = 2 | 42,538 | 141.7 μs | 170.2 μs | 2.0× | 12 / 6 / 7 / 7 % |
| H_iter = 4 | 26,252 | 87.4 μs | 105.0 μs | 3.3× | 25 / 12 / 12 / 13 % |
| **H_iter = 8** | 18,978 | **63.2 μs** | 75.9 μs | **4.5×** | 25 / 23 / 24 / 25 % |
| H_iter = 16 | 23,206 | 77.4 μs | 92.8 μs | 3.7× | 25 / 46 / 54 / 52 % |

**63 μs/token, 4.5× vs H100 PCIe, 2 MB in BRAM, Alveo U55C, 9.96 W on-chip — all four verify.** H_iter = 16 *regresses* because the wider datapath inflates the pipeline interval from ~2,106 to ~6,300 cycles, outweighing halving the iteration count. BRAM plateaus at 25% once the head dimension is fully partitioned.

Energy:

| Platform | Power | Latency | Energy/token |
|---|---|---|---|
| H100 PCIe | 350 W (board TDP) | 285 μs | **99.8 mJ** |
| H_iter = 2 (implemented) | **9.96 W** (Vivado on-chip est., 263 MHz) | 161.7 μs | **1.61 mJ** |
| H_iter = 8 | ≤ 150 W (board TDP) | 63.2 μs | ≤ 9.5 mJ |

The 9.96 W decomposes as 6.54 W dynamic (BRAM 1.67, routing 1.82, clocks 1.18, logic 1.03, DSP 0.85) + 3.42 W static. **62×** is the headline (1.61 vs 99.8 mJ), and the abstract's "up to 60×" is consistent.

> [!warning] The 60× is FPGA *on-chip estimate* vs GPU *board TDP*
> The paper says so itself, and gives the apples-to-apples figure in the same paragraph: **"Even assuming the full 150 W board TDP, energy efficiency ranges from 7.6× to 10.5×."** That 7.6–10.5× is the number to quote in a like-for-like comparison; 60× compares a post-implementation on-chip power estimate at 12% utilisation against a datasheet board TDP.

## Three things that qualify the headline

**1. The 63 μs / 4.5× configuration does not place and route.** Only H_iter = 2 was implemented, reaching 263 MHz (161.7 μs → a realised **1.76×**, derived). H_iter = 4 meets post-synthesis timing with an identical 2.370 ns critical path but **fails routing**: at 77% SLR-level BRAM and 39% LUT it saturates SLR0, spills into SLR1, and produces **88,725 unroutable signals**. The paper is candid that H_iter ∈ {8, 16} "would require explicit multi-SLR floorplanning or a larger device". So 63 μs and 4.5× are **post-synthesis cycle-count estimates × clock period**, not hardware measurements; only 2.0×/1.76× has actually been built.

**2. More than half the best config's latency is host input transfer.** T_load is 8,800–10,600 cycles of AXI loading. At H_iter = 8 that is 10,554 of 18,978 cycles — **~56% of the 63 μs is spent moving 48.5 KB of q/k/v/gates onto the card** (derived), because the state is persistent but the *tokens* are not. This transfer *is* inside the reported number, which is more honest than paper 1. What is *not* inside it is the per-token kernel launch itself: the host "invokes the kernel once per token" over PCIe, and PCIe launch overhead is typically tens of microseconds — plausibly comparable to the whole compute time.

**3. The GPU baseline is a PyTorch reference implementation, and its number is nowhere near its own roofline.** *Derived:* the H100 must move 4,194,304 B of state + 49,664 B of tokens = 4.24 MB at 2.0 TB/s, i.e. **~2.1 μs** of pure HBM time. The measured baseline is **285 μs — about 134× above that bound.** The gap is therefore dominated not by the HBM round trip the paper indicts, but by eager-mode Python/PyTorch launch overhead across 32 sequentially-processed heads. The paper acknowledges optimised kernels (FlashInfer) exist and never benchmarks against them. A fused CUDA kernel would close most of the 4.5×; the *energy* advantage would survive far better than the latency one.

## Weights or state? — and what that means

**Only state.** Concretely, the board holds 2 MB of recurrent state for one layer and nothing else. Consequences:

- It is a **coprocessor for the GDN recurrence**, in the same category as a crypto or codec offload block — not a serving engine comparable to [[vllm]] or [[llama-cpp]].
- **Every layer needs its own host round trip.** *Derived:* Qwen3-Next has 36 GDN layers; at 63.2 μs each that is **~2.27 ms/token ≈ 440 tok/s** for the GDN layers *alone*, before the 12 sparse-attention layers, the MoE FFN, all projections, and 36 PCIe kernel launches. A single H100 serves Qwen3-Next far faster than that end-to-end. The per-layer 4.5× therefore **does not compose into a system-level win** in the design's current form.
- To become a drop-in engine it would need the projections and FFN/MoE weights resident too — but a 2 MB state fits in 17.6 MB of BRAM precisely *because* it is tiny, and Qwen3-Next's weights are three to four orders of magnitude larger. They would live in the U55C's HBM2e at 460 GB/s, i.e. **4.3× less bandwidth than the H100 the paper is beating**. The persistent-state trick does not generalise to weights; that is the structural limit of the result, not an engineering to-do.

The honest reading: this is a **strong existence proof about a bottleneck**, not a serving proposal. It establishes that GDN decode's memory-boundedness is removable in principle and that ~10 W is enough silicon to do the recurrence — a useful data point for edge and embedded settings ([[edge-inference]]) where a small hybrid model's whole weight set might actually fit on-card.

## Limitations

- **Single GDN layer, state only; no weights, no prefill, no attention layers, no MoE** (all listed as future work).
- **The headline configuration was never implemented** — H_iter = 4 already fails routing.
- **GPU baseline is unoptimised PyTorch**, ~134× off its own memory-bound roofline (derived).
- **Energy headline mixes on-chip estimate with board TDP**; the like-for-like figure is 7.6–10.5×.
- **FP32 only**, batch 1 only, one model shape (Qwen3-Next's h_v = 32, d = 128) — batching, which is exactly what makes GDN decode compute-bound on GPU anyway, is not studied.
- **No accuracy evaluation at all** — correctness is C/RTL co-simulation against a golden reference, which is the right check for a bit-exact FP32 datapath but means there is no model-quality result of any kind.

## Related

- [[linear-attention]] — Gated DeltaNet sits on this page's delta-rule → gated → KDA lineage; this source adds the decode-time [[arithmetic-intensity]] consequence of the fixed-size state.
- [[qwen3.8-flash-next]] — the 75% / 3:1 GDN ratio the paper cites; the wiki's own layer count (36 of 48) corroborates it exactly.
- [[kimi-linear-kimi-team-2025]] · [[kimi-k3-moonshot-2026]] · [[flash-kda-moonshot-2026]] — KDA is GDN plus channel-wise gating, so this accelerator's datapath is one gate-shape away from applying to Moonshot's hybrids too.
- [[state-space-model]] — Mamba/Mamba-2 fall in the same sub-1-FLOP/B regime; GDN is described here as a strict superset of Mamba-2.
- [[kv-cache]] — the thing GDN removes, and the reason its state must instead be round-tripped whole every token.
- [[llm-inference-roofline-yuan-2024]] · [[arithmetic-intensity]] — the ridge-point argument; 0.87 vs 25.6 FLOP/B is the whole paper in two numbers.
- [[speculative-decoding]] — the other standard answer to memory-bound batch-1 decode; orthogonal to this one and not combined here.
- [[a-hippocampus-for-linear-attention-cui-2026]] — bolts exact memory onto a GDN state; would change the 2 MB budget this design depends on.
- [[fast-prefill-fpga-sparse-attention-jayanth-2026]] — same lab (Prasanna group, USC), the prefill half of the story.
- [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]] · [[hbm-is-not-all-you-need-wei-2026]] — the phase-vs-hardware framing.
- [[fpga]] · [[hardware-accelerator]] · [[edge-inference]] — no pages yet.
- [[nvidia]] · [[qwen]] · [[vllm]] · [[llama-cpp]] — baseline vendor, model family, and the serving engines this is *not* a substitute for.

## References

- [arXiv:2603.05931](https://arxiv.org/abs/2603.05931)
- Gated DeltaNet: Yang, Kautz & Hatamizadeh, [arXiv:2412.06464](https://arxiv.org/abs/2412.06464) — the recurrence implemented, and the source of the NVLabs PyTorch reference used as GPU baseline.
- DeltaNet: Yang et al., NeurIPS 2024 — the delta rule.
- Mamba-2 / SSD: Dao & Gu, ICML 2024 — GDN is a superset.
- Qwen3-Next: Qwen Team, 2025 — the 3:1 hybrid layer ratio.
- FlashInfer: Chen et al., MLSys 2025 — the optimised kernels cited but not benchmarked against.
