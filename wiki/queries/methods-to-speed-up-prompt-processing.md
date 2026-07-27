---
title: "Methods to Speed Up Prompt Processing (prefill / pp)"
type: query
created: 2026-07-27
updated: 2026-07-27
sources:
  - llm-inference-roofline-yuan-2024.md
  - flash-attention-dao-2022.md
  - sglang-radixattention-zheng-2023.md
  - paged-attention-kwon-2023.md
  - sarathi-chunked-prefill-agrawal-2023.md
  - distserve-zhong-2024.md
  - llmlingua-2-pan-2024.md
  - minimax-sparse-attention-lai-2026.md
tags:
  - 2026
status: complete
importance: high
---

# Methods to Speed Up Prompt Processing (prefill / `pp`)

Prefill and decode have **opposite bottlenecks**, so most "make the LLM faster" advice is decode advice and does nothing here. Measured per-layer arithmetic intensity on Llama-2-7B ([[llm-inference-roofline-yuan-2024]]):

| Phase | Arithmetic intensity | Bound |
|---|---|---|
| **Prefill** (q/k/v/o, FFN) | **1,024 – 1,215** OPs/byte | **compute** |
| Decode | **~1** | memory bandwidth |

A ~1000× gap, against a hardware ridge point of ~200 OPs/byte. So there are exactly three moves: **skip the work, shrink the work, or make the work cheaper per FLOP.**

> [!note] One important refinement
> Prefill is compute-bound *at the roofline level*, but the **attention kernel** is memory-bound, because standard attention materializes an N×N matrix in HBM. That is why [[flash-attention]] speeds up prefill despite prefill being "compute-bound" — the two claims live at different levels of the stack.

## Tier 1 — Skip it: [[prefix-caching]]

The highest-leverage lever, because reused prefill is *free*, not merely faster. Agent loops are the ideal case: system prompts, tool definitions, [[agent-skills|skill]] preambles and history are re-sent byte-identical every turn.

- **[[sglang-radixattention-zheng-2023|RadixAttention]]** — automatic, exact, zero accuracy cost. **6.4× throughput / 3.7× lower latency**, hit rates **50–99%**, overhead when nothing is reusable **<0.3%**. Production Chatbot Arena: **52.4–74.1%** hit rate, **1.7× average first-token latency reduction**.
- **[[prompt-cache-gim-2023|Prompt Cache]]** — reuses named modules at *arbitrary* positions (not just a prefix). **900 ms → 90 ms** TTFT on RTX 4090 / Llama2-7B / 3K ctx. Costs accuracy where cross-module reasoning matters: Passage Retrieval **7.50 → 4.25**.
- Ancestry: [[paged-attention]]'s ref-counted copy-on-write blocks already gave **3.58×** with a 5-shot shared prefix.

**Practical rule:** cache hit rate is the metric. Put invariant content *first* — everything before the first varying byte is cacheable; a timestamp at the top of a system prompt destroys the entire cache.

## Tier 2 — Shrink it

**[[prompt-compression]]** removes tokens before they reach prefill. [[llmlingua-jiang-2023|LLMLingua]] hits **20× on GSM8K for ~1.5 points**, but **−13.2 points on BBH at 7×** — ratios do not transfer across task types. [[llmlingua-2-pan-2024|LLMLingua-2]] makes the compressor cheap (**0.4 s** overhead) and task-agnostic, so compressed documents stay cacheable.

> [!warning] Always check the compressor's own cost
> Selective-Context's compression pass costs **15.5 s** — more than the entire uncompressed inference. A compressor that isn't dramatically cheaper than the model is a **net slowdown**, whatever its ratio.

Evidence prompts are highly compressible: [[can-lms-actually-retrieve-in-context-gollapudi-2026|BlockSearch]] finds the **top 0.1% of context tokens carry 50–80% of the relevance mass**. [[recontext-recursive-evidence-replay-zhao-2026|ReContext]] uses model-internal attention as the selection signal (+24.6% relative, ~1.4× runtime — an accuracy play, not a speedup).

Architecturally, [[agent-skills|progressive disclosure]] is a prefill optimization: tokens never loaded never get prefilled.

## Tier 3 — Cut the quadratic term

**[[flash-attention]]** — exact, not approximate. Does **more FLOPs** (66.6 → 75.2 GFLOPs) while cutting HBM traffic **40.3 GB → 4.4 GB** and runtime **41.7 → 7.3 ms**; memory goes quadratic → linear. v2 reaches **73% of A100 peak**; v3 adds Hopper asynchrony for **740 TFLOPs/s**. This is table stakes, not an optimization.

**[[minimax-sparse-attention-lai-2026|Sparse attention]]** — the approximate route, and the largest pure-prefill number in the wiki: **28.4× less attention compute at 1M context** and **14.2× prefill speedup** on H800, matching dense GQA quality.

**Subquadratic architectures** — [[state-space-model|SSM]]/linear attention; [[a-hippocampus-for-linear-attention-cui-2026|HOLA]] adds a bounded exact cache to recover what the recurrent state forgets.

## Tier 4 — Lower the compute roof (precision)

Only quantization that also quantizes **activations** helps, because only that runs on low-precision tensor cores:

| Scheme | Prefill effect |
|---|---|
| **FP8 / INT8 (W8A8)** | ~2× tensor-core throughput; **~30% TTFT improvement** on H100 |
| **W4A4** ([[quarot-ashkboos-2024\|QuaRot]]) | **up to 3.33× prefill speedup** (batch 64, seq 2048) |
| **W4A16 weight-only** | **2–12% *slower*** — dequantization is added work on the critical path |

See [[quantization-performance]] for the full treatment.

## Tier 5 — Systems and scheduling

**[[paged-attention]]** raises effective KV utilization from **20–38% → 96.3%** for **2–4×** throughput — but via batching, not latency; its kernel is actually **20–26% slower**.

**Phase disaggregation** ([[distserve-zhong-2024|DistServe]], [[splitwise-patel-2023|Splitwise]]) runs prefill and decode on separate GPUs so the two SLOs stop fighting: **2.1× rps/GPU**, **7.4× more requests**, **5.7×** on TTFT-bound code completion, with KV transfer costing **<0.1%** of latency.

## What does *not* speed up prefill

Three traps, all of them things that genuinely help decode:

1. **Weight-only quantization (W4A16, Q4_K_M).** A memory-bandwidth optimization. Measured **2–12% slower** prefill on Apple Silicon.
2. **[[grouped-query-attention|GQA]].** The paper deliberately excludes encoder self-attention because *"encoder representations are computed in parallel, and memory bandwidth is therefore generally not the primary bottleneck"* — that parallel regime **is** prefill. GQA shrinks the KV cache; its prefill benefit is only indirect (freed HBM → larger batch).
3. **[[chunked-prefill]].** Commonly misfiled as a prefill optimization; it is the opposite. SARATHI *spends* prefill — **~5× slower at chunk 64**, ~20% loss at chunk 256 — to gain up to **10× decode throughput**: *"even a 5x overhead in prefills is acceptable if the decodes can be optimized by 2x or more."* It helps *other* requests' TTFT by reducing head-of-line blocking, not the chunked request's own.

Also: [[speculative-decoding]] targets the sequential decode bottleneck; prefill is already parallel.

## Measurement caveats

- **Report `pp` at a stated prompt length** (`pp512`, `pp4096`) — prefill cost moves with sequence length and batch, not model size.
- **Separate TTFT from prefill throughput.** TTFT includes queueing and is what users feel.
- **On CPU, kernel maturity beats physics.** [[which-quantization-kurt-2026|Kurt 2026]] found `pp512` **non-monotonic and uncorrelated with bit width**: Q4_0 fastest at 97.35 t/s, higher-precision Q5_1 only 45.98, Q3_K_S **28% slower than F16**. Benchmark your actual format and backend.

## Recommended order of attack

1. **Prefix caching** — usually the largest win, no accuracy cost, and free if you already run vLLM/SGLang. Restructure prompts so invariants come first.
2. **FlashAttention** — ensure it's actually enabled; it is table stakes.
3. **FP8/INT8** if your hardware has the tensor cores.
4. **Sparse attention** if contexts are genuinely long (≥100K).
5. **Prompt compression** only after validating the ratio on *your* task, and only if the compressor is cheap.
6. **Disaggregation** at multi-GPU serving scale, when TTFT and TPOT SLOs conflict.

## References

- [[llm-inference-roofline-yuan-2024]] · [[flash-attention]] · [[prefix-caching]] · [[prompt-compression]] · [[chunked-prefill]] · [[paged-attention]] · [[grouped-query-attention]] · [[quantization-performance]] · [[minimax-sparse-attention-lai-2026]]
