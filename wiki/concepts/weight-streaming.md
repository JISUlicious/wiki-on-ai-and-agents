---
title: Weight Streaming
type: concept
created: 2026-09-10
updated: 2026-09-10
sources:
  - llm-in-a-flash-alizadeh-2023.md
  - neuroprefetcher-delta-prefetching-dhar-2026.md
  - ssd-offloading-moe-weights-considered-harmful-kyung-2025.md
  - flash-moe-ssd-expert-streaming-woods-2026.md
  - omlx-specprefill-github-2026.md
status: complete
importance: high
---

# Weight Streaming

Running a model whose weights do not fit in DRAM by leaving them on flash storage and reading, per token, only the slice that token needs. Named and first made practical by Apple's [[llm-in-a-flash-alizadeh-2023|LLM in a flash]] (2023); by 2026 it is how a 397B-parameter model runs on a 48 GB laptop.

It is distinct from two things it is often confused with. **Offloading** in the [[zero-offload-ren-2021|ZeRO-Offload]] / FlexGen sense moves whole tensors to CPU RAM or disk and pulls them back on a fixed schedule. **Demand paging** — what [[llama-cpp]] does when a GGUF is memory-mapped and larger than RAM — lets the OS fault pages in as they are touched, with no knowledge of what the next token needs. Streaming means the runtime decides *what* to read and *when*, ahead of use.

## The arithmetic that governs everything

Per-token decode time is bounded below by `bytes read this token / storage bandwidth`. On a Mac the relevant numbers are roughly:

| Tier | Bandwidth | Relative to unified memory |
|---|---|---|
| Unified memory, M4 → M2 Ultra | 120 → 800 GB/s | 1× |
| NVMe SSD, sequential rating | ~5–7 GB/s (M1 Max measured 6.1) | 20–100× slower |
| NVMe SSD, **random reads at streaming granularity** | 1.25 GB/s at 32 threads on the M1 Max, 2.25 with 32 KiB bundling (2023); 1.3 GB/s achieved on a 7.4 GB/s-rated 990 PRO (2026) | **50–600× slower** |

The third row is the one that binds. Streaming reads scattered neuron rows or expert slabs, not a contiguous file, and both measured systems reach only 17–37% of the drive's rating. Making reads larger and contiguous — row-column bundling in 2023, expert-sized slabs in 2026 — is worth as much as any other optimization.

So streaming is only viable if the per-token working set is a small fraction of the model — and that fraction has to be *known before the read*, or the prefetch is guessing. This is the same [[arithmetic-intensity]] question the rest of the wiki's inference pages turn on, with SSD as one more tier below HBM and DRAM.

## Three sources of sparsity, three generations

**1. ReLU [[activation-sparsity|activation sparsity]] (2023).** In a ReLU FFN most intermediate activations are exactly zero, so most up-projection columns and down-projection rows are dead for a given token. A low-rank predictor (~1.25% of model size) guesses which neurons fire; *windowing* keeps the union of the last 4–5 tokens' active neurons in DRAM and loads only the delta — **2.4% of FFN neurons per token on OPT-6.7B**; *row-column bundling* stores each neuron's up-column and down-row contiguously, doubling the read chunk to 32 KiB and lifting random-read throughput from 1.25 to 2.25 GB/s. "Twice the DRAM" means, concretely, that the model is allowed about half its own footprint. Against a naive full reload from flash every token, OPT-6.7B runs **4.8× faster on an M1 Max CPU and 26× on an RTX 4090** (the abstract's "4–5× / 20–25×" is earlier-version wording); against the paper's own half-resident baseline the derived wins are ~3× and ~13×. **It does not transfer to SiLU/GELU models unmodified**: Falcon, Phi-2 and Llama 2 had to be *relufied*, at 1–3 MMLU points each, and the paper says plainly that swapping Swish for ReLU in a SwiGLU FFN does not by itself yield sparsity. It is also candid that energy per token is *higher* than DRAM-resident inference, and its Appendix E names MoE as the natural next target.

**2. Lossy activation thresholding (2026).** Modern open models use SiLU or GELU; nothing is exactly zero. [[neuroprefetcher-delta-prefetching-dhar-2026|NeuroPrefetcher]] (ICPP 2026) recovers sparsity by keeping only the largest-magnitude activations per layer, predicts the next token's active neurons with a predictor 2.86% the size of the model, and issues explicit NVMe reads for the delta only — its measured insight is that **82–85% of active neurons persist from one token to the next**. On an NVIDIA Jetson AGX Orin, Mistral-7B FP16 at an 11.9 GiB budget runs at **3.22 tok/s against 0.34 for llama.cpp**, 7.9–12× across budgets. Three things to hold onto. It is **not [[apple-silicon|Apple Silicon]]**. The win is fewer bytes, not faster bytes — achieved NVMe read is only 1.3 GB/s on a drive rated 7.4. And **the speedup is bought with accuracy**: at the 9.5× point Mistral keeps roughly 88–90% of dense HellaSwag with perplexity 8 against 5.3, and at the 12× points about 68–73%; Llama-3-8B fares worse everywhere. The baseline is FP16 `mmap` thrashing, which nobody deploys, and no quantized model that simply fits is compared.

**3. MoE routing (2026, exact).** A [[mixture-of-experts]] model activates *k* of *N* experts per layer, and the router names them *before* the expert weights are needed. That gives the runtime an exact read list with no predictor and no accuracy change: as the oMLX pull request puts it, a cache miss changes *when* an expert's weights are read, never *which* expert runs, so the entire cost is latency. This is why every Mac implementation below streams experts, and why the technique became practical in the same year that Qwen3.5-397B-A17B, DeepSeek-V4-Flash, [[gemma|Gemma]] 4 26B-A4B and GLM-5.2 shipped as MoE.

## Implementations on Apple Silicon

The catalyst was [[flash-moe-ssd-expert-streaming-woods-2026|flash-moe]] (2026-03-18), a pure C/Metal engine that ran Qwen3.5-397B-A17B on a 48 GB M3 Max at 4.36 tok/s. Everything below followed it, except `mlx-moe`, which predates it by six weeks and went unnoticed.

Every number here is a README self-report as of 2026-09-09. None has been independently measured.

| Project | Base | Demonstrated | Decode | Status |
|---|---|---|---|---|
| flash-moe (danveloper) | C / ObjC / Metal | Qwen3.5-397B-A17B, 48 GB M3 Max, 5.5 GB resident | 4.36 tok/s (4-bit) | 4,090★, frozen after one day |
| flash-moe (Anemll fork) | same | same model, 128 GB M5 Max | 9.5 / 12.9 / 14.5 tok/s (4-bit / Q3 / 2-bit) | 222★, to 2026-03-24 |
| TurboFieldfare | Swift + Metal, MLX weight format, Gemma 4 26B-A4B only | ~2 GB resident on any M-series | 5.1–6.3 tok/s on an 8 GB M2 Air; 31–35 on a 24 GB M5 Pro | 6,681★, active |
| SwiftLM | Swift, mlx-swift-lm, `--stream-experts` | DeepSeek-V4-Flash 126 GB on a 64 GB M5 Pro, ≤17 GB RAM | 4.65 tok/s at 512 ctx | 760★, active |
| oMLX PR #2595 | Python, oMLX | gemma-4-26b in 4.6 GB instead of 14.2 | not stated | **open, unmerged** |
| mlx-moe (mu-hashmi) | Python, patches mlx-lm `SwitchGLU` | Qwen3-Coder-Next 46 GB on a 32 GB Mac | 6–23 tok/s | 6★, Feb 2026, dormant |
| mlx-moe-offload (huckiyang) | Python, `OffloadedSwitchGLU` | 315 GB 2-bit on a 192 GB Mac | **0.03–0.4 tok/s** when the store exceeds page cache | 8★, Jul 2026 |
| deepseek-v4-flash-mlx | Python, from-scratch MLX engine | ~100B mxfp4 on a 48 GB M5 Pro | 4.5–5 tok/s | 3★, one day |
| TensorFold | Python MLX, layer streaming | GPT-OSS | not stated | 40★, Jun 2026 |
| mlx-flash (matt-k-wong) | Python MLX, **dense** streaming per the 2023 paper | Nemotron-30B 17.8 GB on a 16 GB Air | **no decode figure**, load time only | 131★, Mar–Jun 2026 |

**Nothing is upstream.** The [[mlx]]-lm feature request (#1438) has 32 comments and no maintainer reply; the [[omlx-specprefill-github-2026|oMLX]] issue (#986) has a working PR that has not been merged. Every project above is third-party, most are one-person, and the two with real adoption — TurboFieldfare and SwiftLM — are Swift, not Python.

## What the measurements actually show

**The KV cache and the expert cache share one RAM budget, and KV wins.** SwiftLM's DeepSeek-V4-Flash run on a 64 GB M5 Pro: **4.65 tok/s at 512 context, 0.32 tok/s at 40K**. As the [[kv-cache]] grows it evicts cached experts, hit rate collapses, and every token goes back to the SSD. Quantizing the KV cache restores it to **4.16 tok/s at 40K**. At long context, KV quantization is not an optimization of streaming — it is the precondition for it.

**Streaming a model that would fit is a pure loss.** SwiftLM on gemma-4-26b, same M5 Pro: **77.5 tok/s fully resident, 10.8 tok/s streamed** — 7× slower for nothing. The technique has exactly one use: models that do not fit.

**Hit rate is the whole game.** `mlx-moe-offload`'s own README reports **0.03–0.4 tok/s** when the expert store exceeds the page cache — the floor you hit when nothing is cached. The Anemll fork's **3.3× gain from M3 Max to M5 Max on the same code** is the same fact from the other side: more RAM for the cache and a faster SSD, no change to compute.

**The headline number hides a model change.** flash-moe cut experts-per-token from 10 to 4 to shrink the read. That is not an engineering optimization, it is a different model, and no project in the table reports the accuracy delta from reducing *k*.

**Prefill is the unsolved phase.** Every project reports decode. Prefill touches nearly every expert in every layer for a long prompt, so it cannot be sparse, and the streamed read is the full expert set. `mlx-moe-offload` notes it must be chunked or a single lazy forward pins every expert at once. No project publishes a prompt-processing speed.

## The energy objection

[[ssd-offloading-moe-weights-considered-harmful-kyung-2025|Kyung, Yun and Ahn]] (IEEE CAL 2025) argue that at datacenter scale the whole approach is a mistake. Their per-bit read energies — HBM3 4.2 pJ, DDR5 9.2, NAND flash 102.4 — put the SSD path at ~26× HBM per byte, and per token that becomes **3.8–12.5× the HBM baseline on Mixtral 8x7B and 4.7–9.8× on DeepSeek-R1**, or 2.1–3.6× a DDR offload. Prefetching over NVLink holds the *latency* penalty to 1.25–1.32× but moves the same bytes, so it cannot touch the joules. Flash would need to fall to roughly 10 pJ/bit — DDR parity — for a 400B MoE on SSD to beat a 70B dense model in HBM across batch sizes.

> [!note] Read the scope before applying it to a laptop
> The paper is an analytical model, not a measurement: the authors' own simulator, **uniform-random** token-to-expert routing, a single decode step, and **no cross-token expert cache** — so the byte-reducing mechanisms every implementation above depends on are absent by construction. The abstract's "~12× for DeepSeek-R1" is actually Mixtral at batch 16; R1 peaks at 9.8×, and there is no batch-1 R1 point. At batch 1 the MoE-on-flash versus dense-in-HBM penalty is **1.14×**, and break-even sits at 0.7× today's flash energy, not 0.1×. On a consumer NVMe at 1–7 GB/s, bandwidth binds long before energy does. The paper's counterfactual is always a model that fits; on a laptop the alternative is that it does not run. Its §VI nonetheless names mobile and edge as the prime future domain once flash reaches ~10 pJ/bit.

## Related

- [[arithmetic-intensity]] — the byte-budget logic, with SSD as the tier below DRAM.
- [[mixture-of-experts]] — the exact-sparsity source that made this practical.
- [[kv-cache]] · [[quantization-performance]] — the competing RAM consumer and the lever that tames it.
- [[llm-memory-and-compute-bottlenecks]] — the DRAM/HBM side of the same hierarchy.
- [[llama-cpp]] — `mmap` demand paging, and why a too-large GGUF "runs" but crawls.
- [[hardware-accelerator]] — the other two Mac paths (FPGA, eGPU) and why neither works; streaming is the one that does.
- [[mlx]] · [[omlx-specprefill-github-2026]] · [[speculative-decoding]]

## References

- [[llm-in-a-flash-alizadeh-2023]] · [[neuroprefetcher-delta-prefetching-dhar-2026]] · [[ssd-offloading-moe-weights-considered-harmful-kyung-2025]] · [[flash-moe-ssd-expert-streaming-woods-2026]] · [[omlx-specprefill-github-2026]]
