---
title: "Methods to Speed Up Prompt Processing (prefill / pp)"
type: query
created: 2026-07-27
updated: 2026-09-10
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

### Tier 1b — Once caching works, the bottleneck moves *upstream of the model*

Every tier on this page acts on model work. [[toktier-exact-stateful-tokenization-zhang-2026|TokTier]] makes the point that Tier 1 succeeding relocates the problem: **prefix caching can only begin once token IDs exist**, and the front end still re-tokenizes the entire request text on every call. Coding agents pay most, because each turn resubmits a long transcript after a small append that can shift token boundaries near the tail.

At high cache hit rates, tokenization grows to a reported **10% → 64% of TTFT**. The fix is stateful *exact* repair — re-tokenize a small window around the append and splice only when a stable-boundary check passes, otherwise widen or fall back — under a hard contract that emitted IDs always match full reference tokenization (**1.5 × 10¹⁰ split checks across 17 tokenizer families, zero divergence**). Repair costs **0.5–1.1 ms from 100K to 3M characters**, flat in context, versus 1548 ms for a serial HuggingFace re-tokenize at the largest shape.

End to end with vLLM: **median TTFT −16% to −34%, P99 −23%** under load.

> [!warning] Three bounds on this lever
> **It is exactly zero in unloaded single-stream benchmarks** — which is why it has been invisible. It **stops applying past the engine's KV capacity**: at four concurrent 95K-token sessions prefix reuse collapses and both channels degrade identically. And content-caching tokenizers win below a ~100K–500K crossover, so this is a crossover, not a blanket win.
>
> One caveat on the headline: the 10%→64% band corresponds to hit rates approaching **0.99**, not to the separately-reported 94.1% fleet aggregate — three different measurements the summary literature tends to conflate.

Useful corollary for the cache-hygiene rule above: token state costs **~16 bytes/token** against gigabytes for KV, and **55% of Claude Code / 42% of Codex inter-turn gaps exceed the 5-minute cache TTL** — so token state can profitably outlive KV state.

## Tier 2 — Shrink it

**[[prompt-compression]]** removes tokens before they reach prefill. [[llmlingua-jiang-2023|LLMLingua]] hits **20× on GSM8K for ~1.5 points**, but **−13.2 points on BBH at 7×** — ratios do not transfer across task types. [[llmlingua-2-pan-2024|LLMLingua-2]] makes the compressor cheap (**0.4 s** overhead) and task-agnostic, so compressed documents stay cacheable.

> [!warning] Always check the compressor's own cost
> Selective-Context's compression pass costs **15.5 s** — more than the entire uncompressed inference. A compressor that isn't dramatically cheaper than the model is a **net slowdown**, whatever its ratio.

Evidence prompts are highly compressible: [[can-lms-actually-retrieve-in-context-gollapudi-2026|BlockSearch]] finds the **top 0.1% of context tokens carry 50–80% of the relevance mass**. [[recontext-recursive-evidence-replay-zhao-2026|ReContext]] uses model-internal attention as the selection signal (+24.6% relative, ~1.4× runtime — an accuracy play, not a speedup).

Architecturally, [[agent-skills|progressive disclosure]] is a prefill optimization: tokens never loaded never get prefilled.

### Tier 2b — Shrink it *inside* the model: [[sparse-prefill]]

The compressor in Tier 2 is a separate model reading text. **Sparse prefill** uses *attention itself* as the selector — training-free, task-agnostic, and (in the positional variants) position-preserving, so survivors keep their place in the document. Three families, orthogonal and composable:

| Family | Selector | Cuts FFN? | Best regime |
|---|---|---|---|
| **[[speculative-prefill]]** | a small draft model's lookahead attention | yes | 8K–128K, large targets |
| **[[token-pruning]]** | the target's own layer ~13 attention | yes | 8K–64K |
| **[[sparse-attention]]** | per-head pattern; every token stays | **no** | **300K+** |

**The decision rule is attention's share of prefill, which grows with N**: ~¼ at 10K, ~70–80% at 128K, >90% past 300K. So below ~64K only token-dropping helps — [[minference-dynamic-sparse-attention-jiang-2024|MInference]] measures **×0.47, slower than dense**, on LongBench-length prompts — while past 300K sparse attention is the only lever that reaches **10× at 1M**.

The numbers that matter: [[speculative-prefill-liu-2025|SpecPrefill]] at 10% keep gives **7.66× TTFT on a 405B target** with the draft costing 2.96% of its FLOPs, retaining 98.5% of LongBench — but RULER *aggregation* collapses 99.7 → 77.9 and summarization loses ~16%. [[probe-and-skip-self-predictive-token-skipping-wu-2026|SPTS]] reaches 2.46× TTFT at 32K on an 8B with 47.80 vs 47.98 dense. Every token-dropping method shares one hazard: **dropped tokens never get KV**, so a later turn asking about them cannot be answered without re-prefilling.

> [!note] On Apple Silicon
> [[omlx-specprefill-github-2026|oMLX]] ships positional SpecPrefill: a Qwen3.5-0.8B draft scoring for a Qwen3.8-27B target at default keep 20% above 8,192 tokens, system prompt always full, 512-token tail always kept. Ceiling ≈ 4× on the conversation portion. **Nothing in the repo measures it** — treat community throughput figures as one machine's report.

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
3. **[[chunked-prefill]]**, *in a multi-request serving context*. Commonly misfiled as a prefill optimization; there it is the opposite. SARATHI *spends* prefill — **~5× slower at chunk 64**, ~20% loss at chunk 256 — to gain up to **10× decode throughput**: *"even a 5x overhead in prefills is acceptable if the decodes can be optimized by 2x or more."* It helps *other* requests' TTFT by reducing head-of-line blocking, not the chunked request's own.

> [!warning] That verdict does not transfer to single-user local inference
> SARATHI's cost is a *scheduling* trade against concurrent requests. With no other requests in flight there is nothing to trade against, and chunking becomes a **memory-bounding** technique that is often outright faster at the right chunk size. Field measurement on an M4 Air (Qwen3.6-35B-A3B, 3-bit MLX, single user, same prompt):
>
> | Chunk | Prefill |
> |---|---|
> | unchunked, 32K | **OOM** — 34 GB attention buffer |
> | 128 | 106 tok/s |
> | **512** | **135 tok/s** |
> | 2048 | 112 tok/s |
>
> Non-monotonic with a clear optimum, so the chunk size is the whole decision. Below it you pay per-chunk launch overhead; above it the attention buffer grows again. Unchunked does not run at all.
>
> The general rule: **chunked prefill costs you when you are scheduling against other requests and helps you when you are bounding memory.** Establish which regime you are in before reading SARATHI's number as a verdict.
>
> Provenance: these tok/s figures come from an earlier peer session; a successor session later found its harness vulnerable to prefix-cache contamination. The OOM and the non-monotonic shape are robust; treat the absolute tok/s as unverified.

Also: [[speculative-decoding]] targets the sequential decode bottleneck; prefill is already parallel. **Do not confuse it with [[speculative-prefill]]** — same word, opposite phase; a draft with an MTP head can serve both, but they are two levers.

## Measurement caveats

- **Report `pp` at a stated prompt length** (`pp512`, `pp4096`) — prefill cost moves with sequence length and batch, not model size.
- **Separate TTFT from prefill throughput.** TTFT includes queueing and is what users feel.
- **On CPU, kernel maturity beats physics.** [[which-quantization-kurt-2026|Kurt 2026]] found `pp512` **non-monotonic and uncorrelated with bit width**: Q4_0 fastest at 97.35 t/s, higher-precision Q5_1 only 45.98, Q3_K_S **28% slower than F16**. Benchmark your actual format and backend.

## 2026 additions (sweep of 2026-09-10)

Eleven sources ingested in one pass; the numbers below are body-verified unless marked.

**The hybrid-architecture rule.** [[uniprefill-block-wise-dynamic-sparsification-fan-2026|UniPrefill]] makes the structural point every retrofit on this page had been missing: on a hybrid linear/full-attention model, **sparse attention can only touch the full-attention layers** — ≤25% of a 3:1 hybrid, and measured ≤1.11× at any length for every sparse-attention method on Qwen3-Next and Gemma-3. **Token pruning that cascades** — drop a token at a full-attention layer and exclude it from every later layer — cuts GEMM in all layers. But it gains *less* on hybrids than on dense models (1.08–1.68× vs 1.21–2.26× TTFT), because dropping a fraction ρ is ρ² on N² attention and only ρ on GEMMs, and a hybrid has few quadratic layers to collect the ρ². And it is **negative below ~16K on hybrids in serving**: −5% / −4% at 4K / 8K on Qwen3-Next at batch 1.

**A third axis: skip model compute, keep every token.** Tier 2 above shrinks the *prompt*; Tier 3 shrinks *attention*. [[pop-prefill-only-pruning-he-2026|POP]] and [[fast-forward-predictive-ffn-sparsity-prefill-gautam-2026|Fast Forward]] shrink the *model* during prefill only. POP is training-free: skip the last third of layers for prompt tokens 1…N−1, still run their pretrained K/V projections, process the last prompt token with the full model — **1.36× at 2K tokens, 1.02× at 32**, −1.7 accuracy points on Llama-3.1-8B. Fast Forward is not: a neuron predictor plus an error-compensation network (~3.6% new parameters), 1.44× at 2–4K falling to 1.20× at 32K, under 6% LongBench loss. Both compose with everything else on this page.

**Kernel numbers are not TTFT numbers.** [[trianglemix-decoding-time-contribution-sparsity-he-2025|TriangleMix]] headlines 15.3× on the attention kernel at 128K; its end-to-end TTFT on Llama-3.1-8B is **1.14× at 32K and 1.47× at 128K**, in §3.4. [[sale-low-bit-estimation-sparse-attention-prefill-ji-2025|SALE]] reports attention-only latency — 1.39× at 8K implies ~1.07× TTFT — and requires INT4 Tensor Cores, so it has no Apple Silicon path. [[native-sparse-attention-yuan-2025|NSA]]'s 9× is likewise a kernel figure with no end-to-end number in the paper. The attention-share rule in Tier 2b is the conversion.

**Two decode-only methods that look like prefill methods.** [[sts-speculative-token-sparsity-yu-2026|STS]] and [[specattn-speculating-sparse-attention-shah-2025|SpecAttn]] use a draft model's attention to sparsify the *target's verification pass* in speculative decoding. STS's default config keeps prefill dense and reports no TTFT; SpecAttn prefills both models densely and is net slower than plain speculative decoding (59.95 vs 68.26 tok/s). Neither belongs on this page except as a warning; see [[sparse-attention]].

**Trained sparse attention is a model choice, not a kernel choice.** [[trained-sparse-attention]] — NSA, MoBA, DeepSeek-V3.2's DSA — puts the selector in the training loop, and a model that ships it has already taken the long-context win. No evidence exists for stacking a retrofit on top.

**Apple Silicon, two hardware facts.** The M5's GPU Neural Accelerators give **3.3–4.1× TTFT over M4** at a 4,096-token prompt and only 1.19–1.27× on generation ([[exploring-llms-mlx-m5-neural-accelerators-apple-2025|Apple's own post]], vendor numbers). And on M4-generation GPUs FP8 matmul is *emulated* at 0.87–0.94× FP16 throughput ([[rigel-metal-tensor-compute-path-m4-max-kumaresan-2026|Rigel]]), so Tier 4 does not apply to M4 Macs.

> [!warning] Field measurement, retracted and corrected the same day (2026-09-10)
> A peer session first reported that prefill throughput on an M4 Air *climbs* with prompt length — 293 → 749 tok/s on Qwen3.6-35B-A3B. **That measurement was oMLX's prefix cache, not prefill.** The harness appended its nonce to the *end* of a fixed filler, so each rung's prompt was a strict prefix of the next, and the server reported 69–87% of tokens served from cache. Re-run with the nonce first and `cached_tokens` asserted zero on every request, same model, 4-bit:
>
> | Prompt tokens | First reported | Actual | Inflation |
> |---|---|---|---|
> | 5,972 | 395 | **403.7** | 1.0× |
> | 11,871 | 534 | **385.1** | 1.4× |
> | 17,342 | 744 | **233.1** | 3.2× |
> | 23,657 | 749 | **145.9** | 5.1× |
> | 35,441 | 379 | **125.3** | 3.0× |
>
> Prefill is **flat at ~380–400 tok/s to ~12K and then declines**; a 35K prompt costs 283 s, not 93 s. A dense-27B ladder ran on the same script and is withdrawn with it. A three-way comparison against a two-Mac 99B cluster is withdrawn as method-inconsistent: the distributed path appears never to have used the cache, so only the single-node runs were helped, and the weights are gone so it cannot be re-run.
>
> **What survives**: the level tracks active parameters, on weaker evidence than first claimed. **What the bug revealed**: prefix caching is the largest prefill lever on this hardware — roughly **4× wall-clock on a repeated prefix, on by default, no configuration** — consistent with a 6.1× figure from an earlier session that this wiki removed for lack of provenance and which now looks real.
>
> Two more results from the corrected harness. **Speculative prefill cannot pay against a 3B-active MoE.** The draft performs a full prefill plus lookahead, so `speedup = 1 / (c_draft/c_target + keep)`, and it wins only if the cost ratio is below `1 − keep`. A dense 4B draft against the 35B-A3B, both cache-defeated, gives ratios of 1.61 / 1.14 / 0.74 / 0.76 at 1.5K / 6K / 17K / 35K — **0.75× / 0.79× / 1.06× / 1.04× at keep 0.20**. The draft degrades with length as well, so the ratio plateaus near 0.75 instead of opening up. A "4B draft for a 35B target" is a category error when the target is sparse: prefill cost tracks *active* parameters on both sides. Against a dense 27B the projected ratio is 0.16–0.26 and the projected gain 2.2–2.8× — projected, not measured. **The ANE path is a regression** on this model: −35% at 6K and −16% at 17K once its memory-guard headroom was fixed.

## Recommended order of attack

> [!note] 2026 amendments to this list
> - **Step 3 (FP8/INT8)**: not on M4-generation Macs — no low-precision datapath; on M5 the Neural Accelerators are the win and need no format change.
> - **Steps 4–5**: on a hybrid linear/full-attention model (Qwen3-Next, Qwen3.6/3.8 hybrids, Gemma-3, Kimi), skip sparse attention entirely and use the token-pruning family — but only from ~16–32K up; below that it is a net loss.
> - **New step 5b**: on a dense full-attention model at ≥2K prompts, POP's prefill-only layer skipping is training-free and worth ~1.3×, and composes with everything above.
> - **Step 5, speculative prefill**: only when the draft is cheap relative to the target's *active* parameters. A dense 4B draft against a 3B-active MoE is measured at 0.75–1.06× — a loss or a wash at every length.
> - **Step 1, prefix caching**: it is also the thing that will corrupt your prefill benchmarks if any two prompts share a prefix. Put the nonce first and assert `cached_tokens == 0`.

1. **Prefix caching** — usually the largest win, no accuracy cost, and free if you already run vLLM/SGLang. Restructure prompts so invariants come first.
2. **FlashAttention** — ensure it's actually enabled; it is table stakes.
3. **FP8/INT8** if your hardware has the tensor cores.
4. **Sparse attention** if contexts are genuinely long (≥100K).
5. **[[sparse-prefill]]** — speculative prefill or token pruning at 8K–128K if the workload is retrieval/QA-shaped and single-turn; skip it for summarization, aggregation, or multi-turn over the same context.
6. **Prompt compression** only after validating the ratio on *your* task, and only if the compressor is cheap.
7. **Disaggregation** at multi-GPU serving scale, when TTFT and TPOT SLOs conflict.

## References

- [[llm-inference-roofline-yuan-2024]] · [[flash-attention]] · [[prefix-caching]] · [[prompt-compression]] · [[chunked-prefill]] · [[paged-attention]] · [[grouped-query-attention]] · [[quantization-performance]] · [[minimax-sparse-attention-lai-2026]]

## Related 2026 sources

- [[cross-model-kv-cache-transfer]] — a fifth 'skip it' route — reuse the *previous model's* KV when routing swaps models mid-conversation; 4–25× vs re-prefill on matched-KV pairs
- [[from-llm-inference-to-agentic-workloads-chang-2026]] — the orthogonal finding: **the model is often not the bottleneck.** Non-LLM components dominate latency in 5 of 10 instrumented applications, and a fixed 5-minute KV TTL against human-paced idle gaps evicts 59.4% of sessions, causing **31.5% of total monetary cost**
- [[hardware-accelerator]] — the silicon answer to the same question — and why there is **no FPGA path on a Mac**: XRT is Linux-only, no Apple Silicon PCIe, and Thunderbolt gives 3–15 GB/s against ~120 GB/s of unified memory
- [[basert-native-metal-inference-apple-silicon-rathnayaka-2026]] — runtime choice is not a prefill lever on Apple Silicon: BaseRT trails llama.cpp on dense prefill in 7 of 8 configs and MLX from 512 tokens up; its MoE prefill lead exists only at short prompts. Engine proprietary; no M5 comparison
- [[above-the-inner-loop-m1-amx-prefill-gemm-bhan-2026]] — CPU-side prefill on M1: 1.44× at 128 tokens, 0.82× at 512 — the AMX path does not scale with prompt length
- [[misa-mixture-of-indexer-sparse-attention-zhou-2026]] — a training-free speedup for models *already* running DSA: 3.82× on the indexer kernel, ~3% of per-token compute, so low-single-digit end-to-end
- [[gated-sparse-attention-shen-2026]] — from-scratch trained sparse attention with element-wise gates; the long-context gain is mostly the gates'
