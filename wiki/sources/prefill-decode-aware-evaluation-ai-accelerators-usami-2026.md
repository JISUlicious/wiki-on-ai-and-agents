---
title: "Prefill/Decode-Aware Evaluation of LLM Inference on Emerging AI Accelerators"
type: source
created: 2026-09-09
updated: 2026-09-09
sources: []
arxiv_id: "2606.17104"
authors:
  - Shun Usami
  - Venkatram Vishwanath
  - E. Wes Bethel
year: 2026
venue: "HPAI4S'26 (Workshop on HPC for AI Foundation Models & LLMs for Science), co-located with IEEE IPDPS 2026"
status: complete
importance: medium
tags:
  - 2026
  - paper
  - hardware
---

# Prefill/Decode-Aware Evaluation of LLM Inference on Emerging AI Accelerators

**Shun Usami** (San Francisco State University), **Venkatram Vishwanath** (Argonne National Laboratory), **E. Wes Bethel** (SFSU / Lawrence Berkeley National Laboratory) — [arXiv:2606.17104](https://arxiv.org/abs/2606.17104), cs.AR / cs.AI / cs.DC, 14 Jun 2026, 8 pages. **Accepted to HPAI4S'26, co-located with IEEE IPDPS 2026.** Used ALCF resources.

## Problem

MLPerf Inference reports aggregate throughput under TTFT/TPOT constraints, and non-GPU accelerators barely participate; LLM-Inference-Bench covers more hardware (including SambaNova SN40L, see [[sambanova]]) but also reports end-to-end numbers only. So nobody has answered the obvious question raised by [[distserve-zhong-2024|DistServe]]-style disaggregation: **if prefill and decode have opposite computational characters, do different *chips* have opposite strengths too?** This paper measures prefill and decode separately, on one model, across a GPU and a dataflow accelerator, and then asks whether you should put each phase on a different machine.

## Setup

Llama2-7B (see [[llama]]) in FP16, chosen because it is *nearly the largest model that fits entirely in GroqRack's SRAM*: 12.56 GiB of weights + up to 2 GiB of [[kv-cache]] (512 KiB per token × 4,096 context) ≈ **14.6 GiB against 15.42 GiB of SRAM**.

| | NVIDIA A100 SXM-40GB | GroqRack (LPU) |
|---|---|---|
| System | single GPU | **72 GroqCards across 9 nodes** |
| SRAM | 40 MiB L2 (hardware cache) | **15.42 GiB total** (230 MB/card, compiler-managed scratchpad) |
| DRAM | 40 GB HBM2 | **none** |
| Peak FP16 | 312 TFLOPS | 13.5 PFLOPS (187.5 TFLOPS/card) |
| Memory BW | 1.6 TB/s | **5.76 PB/s** (80 TB/s/card) |
| Software | [[vllm]] v0.12.0 | GroqFlow SDK, Groq-supplied binary |

Workloads are synthetic with L_in = L_out ∈ {100, 200, 400, 800, 1600}; batch B ∈ {1…32} on A100, **B = 1 only** on GroqRack. Tokenisation excluded; warmup with unique tokens to defeat [[prefix-caching]]. **No output-quality evaluation of any kind** — performance only.

The execution-model difference is the whole story. On A100, prefill is one batched forward pass over all input tokens. On GroqRack, the LPU has **no branch instructions**, so a compiled binary must run at exactly the batch size it was compiled for; the binary Groq supplied takes a single request and **one token per invocation**, which means *prefill executes token-by-token, exactly like decode*. And each token's sampling happens on the **rank-0 host CPU followed by an MPI broadcast** — a host round trip per token, at rack scale.

## Measured results

| Metric | NVIDIA A100 | GroqRack |
|---|---|---|
| **Prefill** | | |
| TTFT (B=1, L_in=100) | **16.8 ms** | 252 ms |
| TTFT (B=1, L_in=1,600) | **103.7 ms** | 4,072 ms (strictly linear) |
| Prefill throughput (peak) | **16,322 tok/s** (plateaus 15–16K at B×L_in ≈ 800–1,600) | 370–397 tok/s (flat) |
| **Decode** | | |
| TPOT (B=1) | 11.88–13.64 ms | **2.98–3.05 ms** (flat across all lengths) |
| TPOT (B=32) | 14.92–**57.90 ms** | N/A (no batching) |
| Decode throughput (B=1) | 73–84 tok/s | **328–336 tok/s** |
| Decode throughput (B=32) | **553–2,144 tok/s** | N/A |

Headline ratios from the conclusion, all of which check out against the table: GPUs win prefill by **up to 39× on latency** (4,072/103.7 = 39.3) and **41× on throughput** (16,322/397 = 41.1); GroqRack wins decode by **~4× on per-token latency** (13.64/3.05 = 4.5; 11.88/2.98 = 4.0). The GPU's decode throughput crosses over GroqRack at **B > 4** — but the advantage collapses with sequence length: at B=32 with 1,600 in / 1,600 out the GPU manages only 553 tok/s, a mere **1.7×** over a GroqRack doing batch 1.

## What paper 3 concludes about prefill vs. decode on non-GPU accelerators

The finding is a clean phase/hardware asymmetry, but the *reasons* matter more than the ratios, and two of the four are not architectural:

1. **GPUs own prefill, and this is architectural.** Prefill is one big matrix–matrix multiply whose effective size is B × L_in; GPUs amortise fixed overhead and approach peak utilisation as that grows. Nothing about a dataflow accelerator changes that.
2. **GroqRack loses prefill for an *implementation* reason, not a hardware one.** The provided binary processes input tokens one at a time, so prefill inherits decode's shape — hence strictly linear TTFT and flat 370–400 tok/s throughput. A batched or chunked prefill binary would change this number substantially. The 39× should be read as *"this build"*, not *"this silicon"*.
3. **GroqRack owns decode latency, and this *is* architectural.** All weights and KV live in explicitly-managed SRAM with no cache hierarchy at all, so latency is set by a static schedule rather than dynamic memory behaviour: TPOT is essentially constant regardless of sequence length. This is the same insight as [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026|persistent on-chip state]] — decode's enemy is the memory hierarchy, and the fix is to not have one.
4. **The GPU's decode degradation is a KV-capacity cliff, precisely diagnosed.** At B=32 with 1,600+1,600, the KV footprint is **102,400 tokens against vLLM's reported HBM allocation of 45,584** — over 2× oversubscribed, so vLLM throttles concurrency and requests queue. That is the mechanism behind the 57.90 ms TPOT and the throughput collapse, and it is a [[paged-attention]]-budget problem rather than a compute one.

The practical conclusion the authors draw: **no accelerator dominates both phases, aggregate metrics hide this, and hardware choice depends on whether you optimise latency or throughput.** GPUs' versatility also makes them the natural overflow capacity for decode when a specialised decode fleet saturates.

## Heterogeneous disaggregation analysis (modelled, not built)

> [!warning] This half of the paper is analytical
> Both platforms were measured with **the KV cache resident on one device for both phases; no cross-device transfer was ever performed**. The disaggregation result is a model built on measured per-phase latencies plus assumed link bandwidths, and the authors say so: "this analysis is based on analytical and hypothetical estimates rather than a fully integrated system."

Transfer cost model: t_transfer = (KV per token) / (bandwidth × 0.8 efficiency), with Llama2-7B at **512 KiB per token**:

| Interconnect | t_transfer/token | Heterogeneous beats A100-only when |
|---|---|---|
| 25 Gbps Ethernet | 0.210 ms | L_out/L_in > **1/52 – 1/42** |
| 100 Gbps HPE Slingshot | 0.052 ms | L_out/L_in > **1/203 – 1/169** |
| 1.8 TB/s NVLink (theoretical bound only) | 0.00036 ms | L_out/L_in > 1/29,300 – 1/24,400 |

So splitting prefill onto the A100 and decode onto GroqRack wins **unless the input is roughly 50× longer than the output** — i.e. only for classification-style prompts that read a huge context and emit a token or two. Against *GroqRack-only*, heterogeneous is unconditionally better with **no break-even point at all**, because the prefill gain (≥ 2 ms/token) always exceeds the transfer cost (≤ 0.21 ms/token).

The authors flag the real blocker honestly: homogeneous GPU clusters have NVLink and libraries like NIXL, but **no standardised mechanism exists for KV cache transfer between a GPU and a non-GPU accelerator**. The break-even condition is also proposed as a live routing rule — decide per request from (L_in, L_out), utilisation, and congestion.

## Limitations

- **Disaggregation is modelled, never measured** — no cross-device KV transfer was executed.
- **One model, one size, one precision** (Llama2-7B FP16), chosen because it happens to fit GroqRack's SRAM; the conclusions do not obviously survive a model that does not fit.
- **GroqRack is limited to B=1** both by the no-branch compilation model and by capacity (a second concurrent request needs 16.56 GiB > 15.42 GiB), so every GroqRack throughput number is a batch-1 number.
- **The Groq prefill result reflects a supplied binary**, not the ceiling of the architecture.
- **L_in = L_out only**, synthetic random tokens, single-request timing — no arrival process, no multi-tenancy, no congestion, no queueing model.
- **No accuracy, correctness, or quality evaluation.** No cost or power analysis either ("we do not consider economic factors").
- Only one non-GPU accelerator is actually measured, despite the plural in the title.

## Related

- [[distserve-zhong-2024]] — the disaggregation formalisation this paper extends from *across GPUs* to *across vendors*; the KV-transfer-overhead framing is inherited directly.
- [[splitwise-patel-2023]] — the other phase-splitting ancestor, which already argued for different hardware per phase; this paper supplies measurements for that argument.
- [[hbm-is-not-all-you-need-wei-2026]] — the same question answered with memory technology rather than vendor, and with a built system rather than a model.
- [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]] — independent confirmation that decode's problem is the memory hierarchy; Groq's answer (put everything in SRAM) and the FPGA's answer (pin the state in BRAM) are the same answer at different scales.
- [[fast-prefill-fpga-sparse-attention-jayanth-2026]] — the counter-case: an accelerator that *does* win at prefill, by attacking the sparse-attention memory pattern rather than the dense matmul.
- [[kv-cache]] · [[paged-attention]] — 512 KiB/token is both the transfer cost and, via vLLM's block budget, the cause of the GPU decode cliff.
- [[chunked-prefill]] — the obvious missing lever on the Groq side, and what would most change its prefill numbers.
- [[llm-inference-roofline-yuan-2024]] · [[arithmetic-intensity]] — the compute-bound/memory-bound split this paper measures empirically.
- [[quantization-performance]] — an orthogonal axis held fixed here (FP16 only), and the one that decides whether a model fits an SRAM-only device at all.
- [[methods-to-speed-up-prompt-processing]] · [[from-llm-inference-to-agentic-workloads-chang-2026]] — the serving-side context; agentic workloads shift the L_out/L_in ratio that this paper's break-even rule turns on.
- [[long-context-llm]] — 4,096-token ceiling here; the KV-per-token cost is what stops the analysis extending.
- [[groq]] · [[hardware-accelerator]] — no pages yet.
- [[nvidia]] · [[sambanova]] · [[vllm]] · [[llama]] — platforms, engine and model.

## References

- [arXiv:2606.17104](https://arxiv.org/abs/2606.17104)
- DistServe: Zhong et al. — the P/D disaggregation formalisation ([[distserve-zhong-2024]]).
- MLPerf Inference (MLCommons) v5.1 datacenter results — the benchmark whose aggregate metrics this paper argues are insufficient.
- LLM-Inference-Bench — broader hardware coverage (incl. SambaNova SN40L) without phase separation.
