---
title: "HBM Is Not All You Need: Efficient Disaggregated LLM Serving across Memory-heterogeneous Accelerators"
type: source
created: 2026-09-09
updated: 2026-09-09
sources: []
arxiv_id: "2606.29986"
authors:
  - Zhixiang Wei
  - Yun Wang
  - James Yen
  - Mingyuan Xia
  - Zhengwei Qi
year: 2026
venue: null
status: complete
importance: high
tags:
  - 2026
  - paper
---

# HBM Is Not All You Need: Efficient Disaggregated LLM Serving across Memory-heterogeneous Accelerators

**Zhixiang Wei, Yun Wang, James Yen, Zhengwei Qi** ([[shanghai-jiao-tong-university|Shanghai Jiao Tong University]]) and **Mingyuan Xia** (UltraRISC) — [arXiv:2606.29986](https://arxiv.org/abs/2606.29986), cs.AR / cs.DC, 29 Jun 2026, CC BY 4.0. Marked "Preprint"; **no venue stated**. System name: **HMA-Serve**.

## Problem

This is [[distserve-zhong-2024|DistServe]] taken one step further, and it is the most directly consequential of the four hardware papers for this wiki. DistServe split prefill and decode across *identical* GPUs; [[splitwise-patel-2023|Splitwise]] split them across *different GPU generations* — but both sides stayed HBM-based, so the HBM premium was paid twice. This paper's observation is that **the premium buys nothing during prefill**:

- HBM is **18% of an A100's manufacturing cost, rising to 45% on a B200**; HBM3e contracts at roughly **$15/GB, about 3× commodity GDDR**, and accounts for 35–45% of an accelerator's build cost.
- Prefill's [[arithmetic-intensity]] scales with prompt length and becomes compute-bound past ~1K tokens. Measured on a single A100 across Qwen3 4B–32B in BF16: **MFU exceeds 70% past 4K while MBU falls below 10% around 1K and under 1% at 16K.** At L = 4K the A100 leaves **96.8–97.2% of its HBM bandwidth idle**, rising above 99% at 16K.

So you are renting the most expensive component on the die and using ~3% of it for half the workload. The proposal — **MemHA**, memory-heterogeneous accelerator serving — is to put prefill on cheap GDDR silicon and decode on HBM. A Tenstorrent Blackhole p150 delivers **664 TFLOPS of BFP8 at roughly $1,300**, an order of magnitude cheaper than an A100 with comparable low-precision throughput.

The catch, and the actual research contribution, is that **the cheapest pairing is necessarily cross-vendor**, which breaks two things single-vendor disaggregation takes for granted: a KV format both ends read natively, and a shared software stack. Tenstorrent emits BFP8 in a **32×32 tiled layout**; NVIDIA decode kernels want BF16 in the paged row-major layout [[flash-attention|FlashAttention]] consumes. Both obvious bridges are bad — dequantising on the producer doubles wire traffic and puts a conversion pass on the prefill critical path; a standalone consumer pass adds a full-tensor HBM read that competes with attention.

The taxonomy the paper draws is worth keeping: **HBM-based** (A100/H100/H200/B200, MI300X, TPU — 2–8 TB/s over 80–192 GB, essential for decode), **GDDR-based** (Tenstorrent Wormhole/Blackhole, NVIDIA L40/L4, the announced Rubin CPX — ~3× cheaper per GB, bandwidth-starved for decode but fine for prefill), and **SRAM-based** ([[groq]] LPU, Cerebras WSE-3 — hundreds of TB/s but only tens of GB, so a 70B model needs sharding that inflates inter-chip cost). That third row is exactly the platform [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026|paper 3]] measured, and the two papers agree on its character.

## Mechanism

Three coordinated mechanisms, all aimed at keeping the cross-vendor seam off the critical path:

1. **Phase-wise quantization.** Prefill runs in Tenstorrent-native **BFP8** (mixed BFP8/BFP4 weights, BFP8 activations and KV); decode stays in **BF16** on the A100. Rationale: prefill is compute-bound and low-precision-tolerant, decode is memory-bound and sensitive to error accumulated over long generations. BFP8 weights speed prefill by a measured **1.1–1.5×** over BF16-uniform (largest at short prompts, where weight-fetch bandwidth dominates) and halve prefill-side weight memory.
2. **Compute–transfer pipeline.** The prefill runtime is *monkey-patched* to expose a per-layer completion event; each layer's KV is gathered into a device-DRAM staging buffer inside the execution trace, then a push kernel using **eight Tensix data-movement cores** writes raw BFP8 tile bytes straight into pinned host memory over **PCIe at ≈6.7 GB/s — about 7× the standard host-pulled readback (≈0.93 GB/s)**. RDMA streams ship each layer while later layers are still prefilling; the NIC reads the pinned buffer directly, keeping the host CPU off the path.
3. **Deferred dequantization.** Raw BFP8 bytes are shipped verbatim (halving wire traffic) and **kept as BFP8 pages in the decoder's HBM**, reconstructed lazily inside the [[paged-attention|paged-attention]] kernel. One fused pass does four jobs: block-float decode (shared exponent + 7-bit mantissa) to BF16, tiled→row-major conversion merging per-chip shards, the post-RoPE key re-layout bridging Tenstorrent's interleaved head-dim order to FlashAttention's half-split order (keys only — values carry no rotary), and the first paged read. The key trick: **BFP8→BF16 is bit manipulation, not floating-point arithmetic**, so it runs on the **integer ALU** that GEMM-heavy decode leaves idle — disjoint hardware from the tensor cores, so it does not contend with decode at all.

## The pairing argument

The roofline partitions the accelerator landscape into three regions, and prefill's and decode's intensity rays fall in different ones:

| Region | Chips | Suits |
|---|---|---|
| Compute-rich, bandwidth-modest | GDDR (Tenstorrent p150, L40, Rubin CPX) | **prefill** |
| Bandwidth-rich | HBM GPUs (A100…B200, MI300X, TPU) | **decode** |
| Bandwidth-extreme, capacity-poor | SRAM (Groq LPU, Cerebras) | decode, if the model fits |
| Both (costly upper-right) | flagship HBM parts | one chip doing both — the expensive default |

And GDDR is not merely adequate — it is competitive on measured wall clock. Qwen3-32B prefill, four-card Tenstorrent mesh vs. a **tuned single-A100 vLLM deployment with CUDA graphs**:

| Prompt length | Result |
|---|---|
| L ≤ 4K (Tenstorrent trace mode, prebaked dispatch graph) | mesh **1.33–1.38× faster** |
| 8K (runtime falls back to eager per-op dispatch) | mesh **1.19× faster** |
| ≥ 16K | A100 reclaims the lead, by **under 1.07×** |

at roughly one-tenth the per-chip price. Note the honest detail: the Tenstorrent advantage shrinks precisely where its runtime leaves trace mode — a software cliff, not a silicon one.

## Measured results

Testbed: **four Tenstorrent Blackhole p150 for prefill + one NVIDIA A100-80GB for decode over a 100 Gb RoCE fabric**, everything served through [[vllm|vLLM]] 0.19.1, on real silicon. [[prefix-caching]] disabled everywhere (the GDDR prefill path keeps none) and the Tenstorrent core clock pinned so idle down-clocking does not distort timing. Baselines: **DistServe-Homo** (1P1D across two A100s with real GPUDirect-RDMA, no host staging — the strongest same-vendor baseline) and **Sarathi-Hetero (Oracle)**, [[chunked-prefill]] colocation across both boxes at whichever split maximises SLO-meeting throughput, which upper-bounds any static colocation policy. Metric: **goodput at 90% SLO attainment**, SLOs set to 5× DistServe-Homo's own no-load single-request latency.

| Model | Trace | In/Out | Prefill layout | TTFT/TPOT SLO | Goodput vs. best baseline | Goodput-per-$ vs. DistServe-Homo |
|---|---|---|---|---|---|---|
| Qwen3-4B | ShareGPT | 512/512 | DP=4 | 0.1 s / 60 ms | **0.53×** (24 vs 45.6 req/s) | **0.8×** |
| Qwen3-8B | ShareGPT | 1024/384 | DP=4 | 0.8 s / 70 ms | **2.3×** | **3.5×** |
| Qwen3-14B | LongBench | 2048/256 | DP=2, TP=2 | 2 s / 120 ms | **3.2×** | **4.8×** |
| Qwen3-32B | arXiv | 4096/64 | TP=4 | 7 s / 240 ms | **1.85×** (1.1× over oracle colocation) | **3.0×** |

Both headline claims — **3.2× goodput and 4.8× goodput-per-dollar** — come from the 14B row and verify exactly. Costing is transparent: an A100 = 1.0, a p150 = **1/12**, so HMA-Serve's box is **1.33 A100-equivalents** against DistServe-Homo's **2.0**. Against the oracle colocation running on the *same* 1.33-cost box, HMA-Serve is 1.2–5.7× better, which isolates the software contribution from the hardware one.

The paper is candid about its loss: **at 4B, homogeneous wins** (24 vs 45.6 req/s), because a single A100 prefills a 512-token prompt in tens of milliseconds — faster than four parallel GDDR chips. The crossover is governed purely by whether prefill saturates one A100; from 8B up, cheap parallel GDDR wins decisively. The 4B cost gap (0.8×) is projected to close to 1.2× with an eight-chip data-parallel prefill, but that is a **projection, not a measurement**.

KV transfer economics, all measured: Qwen3-32B at 8K is ≈2 GB in BF16 (**1 GB in BFP8**); over ~10 GB/s effective RDMA a monolithic transfer costs **100–200 ms, 10–15% of prefill latency**, all charged to TTFT by a naive schedule. Pipelined at 92.3 Gb/s, per-request egress drops to **1.5–49 ms against ~0.9 s of prefill**. The authors note the overlap saves little for a *single* request — its value is letting egress hide behind the *next* request under sustained load, which is where goodput is set. The fused dequant kernel rebuilds a whole request's KV in **~1.5 ms vs 72 ms** for naive elementwise reconstruction, and decode throughput stays **within 1%** of a dequant-free baseline.

**Quality** (MATH500 / AIME24 / AIME25):

| Model | Precision | MATH500 | AIME24 | AIME25 |
|---|---|---|---|---|
| Qwen3-32B | BF16 | 97.4 | 85.3 | **86.7** |
| Qwen3-32B | BFP8 both phases | 97.3 | 83.3 | 84.7 |
| Qwen3-32B | **HMA-Serve** | 97.4 | **86.7** | 85.3 |
| Qwen3-8B | BF16 | 93.7 | 75.5 | 67.8 |
| Qwen3-8B | BFP8 both phases | 94.1 | 76.5 | **75.3** |
| Qwen3-8B | **HMA-Serve** | **94.4** | **76.7** | 66.7 |

> [!warning] The quality narrative does not fully hold on the 8B row
> The paper's claim is that "fully quantizing both phases (BFP8) costs accuracy on the harder AIME problems." That holds for **Qwen3-32B** (83.3/84.7 vs 85.3/86.7). It **does not hold for Qwen3-8B**, where full BFP8 beats BF16 on all three benchmarks and beats HMA-Serve on AIME25 by **8.6 points (75.3 vs 66.7)** — the largest single gap in the table, and in the direction opposite the thesis. On 8B AIME25 the fully-quantized pipeline is simply the best configuration measured. The headline "no measurable loss on generation-quality benchmarks" is defensible; the mechanistic story that BF16 decode is what *preserves* quality is not supported by half the data, and the spread is more consistent with AIME's small-sample variance (30 problems per year) than with a precision effect either way. See [[quantization-quality]] — this is exactly the regime where benchmark noise exceeds the quantization signal.

## Relation to DistServe

Same core idea, one axis further out. [[distserve-zhong-2024|DistServe]] established that prefill and decode should run on **separate instances** to remove interference, and paid a KV transfer to do it — but assumed a homogeneous pool of identical HBM GPUs. [[splitwise-patel-2023|Splitwise]] loosened that to different GPU *generations* (older H100s prefilling for newer H200s), exploiting decode's greater bandwidth sensitivity, but both pools remained HBM, so the cost premium was paid on both sides. HMA-Serve keeps DistServe's scheduling structure — and uses DistServe-Homo as its calibration baseline and SLO reference — while changing what the two pools are made of: **disaggregation across heterogeneous *memory technology* and across *vendors*, not across GPUs.**

Two consequences follow that DistServe never had to face, and they are the paper's real content:

- **The KV path is longer than a network hop.** Cross-vendor egress must cross a device-to-host DMA before it reaches the fabric — hence the custom 6.7 GB/s Tensix push kernel. DistServe-Homo uses GPUDirect-RDMA with no host staging at all.
- **The two ends disagree on number format and memory layout.** This is structural, not a quirk of these two chips: every vendor's matrix engine peaks only at its own native low precision (NVIDIA FP16→FP8→NVFP4 across A100/H100/B200; AMD FP8→MXFP4), and those formats rarely agree across vendors *or generations*. Any MemHA pairing spans at least two native formats, so a single global precision necessarily parks one side off its efficient point. The paper's move — treat this as a *lever* (phase-wise precision) rather than a compatibility tax — is what distinguishes it from **SPAD** (Zhang et al. 2025), which proposed exactly this GDDR-prefill/HBM-decode pairing but only in **simulation**, and from **Mix-Quant** (Lu et al. 2026), which does phase-asymmetric precision (NVFP4 prefill, BF16 decode) on a **single unchanged Blackwell GPU**.

## Host integration

The strongest of the four papers here. Everything is PCIe cards on a real cluster: four p150 prefill cards, one A100-80GB decode card, 100 Gb RoCE between hosts. The device-to-host path is measured (**6.7 GB/s** achieved vs **0.93 GB/s** for the naive readback), the fabric is measured (**92.3 Gb/s**), and both are on the reported critical path rather than excluded from it. **No power figures are given anywhere** — the cost argument is entirely capital cost ($1,300/card, 1/12 of an A100), not energy — and GDDR capacity per p150 is never stated.

## Limitations

- **The 8B AIME25 quality result contradicts the paper's own mechanism claim** (above), and quality is evaluated on only three math benchmarks — no long-context, coding, or instruction-following evaluation despite LongBench and arXiv being used as *load* traces.
- **One prefill vendor, one decode vendor, one fabric.** The cost model (p150 = 1/12 of an A100) is list-price arithmetic that ignores power, cooling, rack space, engineering effort, and the fact that one side required monkey-patching a closed runtime.
- **4B genuinely loses** on both goodput and cost; the fix is a projection to eight chips, not a measurement.
- **Prefix caching is disabled everywhere** because the GDDR path has none — a real deployment penalty, since [[prefix-caching]] is one of the largest production wins available and MemHA forfeits it on the prefill side.
- **Fixed input/output lengths per regime** under Poisson arrivals; no mixed-length or multi-turn traces, and the whole argument turns on the L_in/L_out ratio.
- **Tenstorrent's advantage is partly a trace-mode artifact** — it degrades at 8K when the runtime falls back to eager dispatch and inverts by 16K.
- Decode is always **a single A100**, so decode-side scaling is never tested.

## Related

- [[distserve-zhong-2024]] — the direct ancestor and the calibration baseline; same disaggregation, homogeneous HBM pool.
- [[splitwise-patel-2023]] — heterogeneous *GPU generations*, still all HBM; this paper's step is heterogeneous *memory technology*.
- [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]] — the same pairing question posed against an SRAM accelerator and answered analytically; this paper answers it on real silicon and reaches a compatible conclusion.
- [[fast-prefill-fpga-sparse-attention-jayanth-2026]] — the other "cheap silicon can do prefill" argument, via sparsity rather than via memory economics.
- [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]] — the decode side of the same split.
- [[chunked-prefill]] — the colocation alternative; the paper reproduces Sarathi-Serve's finding that chunked-prefill colocation matches or beats 1P1D up to 14B, and only loses at 32B under a tight interactive TPOT bound.
- [[kv-cache]] · [[paged-attention]] — BFP8 pages held raw in HBM and reconstructed inside the paged-attention kernel is a genuinely new use of the paged abstraction.
- [[prefix-caching]] — sacrificed by the GDDR prefill path.
- [[flash-attention]] — its expected key layout is one of the four things the fused dequant kernel has to bridge.
- [[quantization]] · [[quantization-performance]] · [[quantization-quality]] — phase-wise precision (BFP8 prefill / BF16 decode) is a new axis for these pages: precision chosen per *phase* and per *vendor*, not per model.
- [[llm-inference-roofline-yuan-2024]] · [[arithmetic-intensity]] — MFU >70% vs MBU <1% is the sharpest empirical statement of the prefill/decode split in any of these four papers.
- [[methods-to-speed-up-prompt-processing]] · [[from-llm-inference-to-agentic-workloads-chang-2026]] — serving context; agentic traffic moves the L_in/L_out ratio that decides whether MemHA pays.
- [[long-context-llm]] · [[longbench]] — the 14B regime where the win is largest is the long-context one.
- [[hardware-accelerator]] · [[groq]] · [[tenstorrent]] — `tenstorrent` has no page yet.
- [[nvidia]] · [[qwen]] · [[vllm]] · [[shanghai-jiao-tong-university]] — vendor, model family, engine, affiliation.

## References

- [arXiv:2606.29986](https://arxiv.org/abs/2606.29986)
- DistServe: Zhong et al., 2024 — [[distserve-zhong-2024]].
- Splitwise: Patel et al., 2024 — [[splitwise-patel-2023]].
- Sarathi-Serve: Agrawal et al., 2024 — chunked prefill, and the relative-SLO convention followed here.
- Mooncake: Qin et al., 2025 — the other disaggregated serving system cited.
- Orca: Yu et al., 2022 — iteration-level continuous batching.
- SPAD: Zhang et al., 2025 — the same GDDR/HBM pairing, in simulation only.
- Mix-Quant: Lu et al., 2026 — phase-asymmetric precision on a single GPU.
- vLLM: Kwon et al., 2023 — [[vllm]], the serving stack on both sides.
