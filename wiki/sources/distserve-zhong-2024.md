---
title: "DistServe: Disaggregating Prefill and Decoding for Goodput-optimized Large Language Model Serving"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - distserve-zhong-2024.md
arxiv_id: "2401.09670"
year: 2024
venue: OSDI 2024
authors:
  - Yinmin Zhong
  - Shengyu Liu
  - Junda Chen
  - Xin Jin
  - Hao Zhang
tags:
  - 2024
  - paper
status: complete
importance: high
---

# DistServe: Disaggregating Prefill and Decoding for Goodput-optimized Large Language Model Serving

**Yinmin Zhong, Shengyu Liu, Junda Chen, Jianbo Hu, Yibo Zhu, Xuanzhe Liu, Xin Jin, Hao Zhang** (Peking University; StepFun; UC San Diego) — [arXiv:2401.09670](https://arxiv.org/abs/2401.09670), OSDI 2024.

## Summary

DistServe's argument is that prefill and decode should not merely be scheduled apart — they should run on **different GPUs**. The two phases have different bottlenecks (prefill is compute-bound; decode does comparable I/O per step while producing one token, so it is memory-bandwidth-bound) and different latency metrics (**TTFT** for prefill, **TPOT**, time per output token, for decode). Colocating them, as every prior system does to share weights and KV cache, causes *prefill-decoding interference*: a long prefill step batched with decodes stretches every decode in that batch. It also couples the resource allocation and the parallelism plan, forcing one plan to serve two workloads with opposite characteristics.

The paper's optimization target is **goodput**: the maximum per-GPU request rate that still meets both the TTFT and TPOT SLOs for a target fraction (usually 90%) of requests. This is the key framing difference from throughput-oriented work — raw tokens/second can be high while goodput is near zero if the SLOs are being missed. Their motivating measurement: serving a 13B model on one A100 80GB with 512-token inputs and 64-token outputs, a colocated system achieves **1.6 rps** at 90% SLO attainment, while a prefill-only instance reaches **5.6 rps** and a decode-only instance **10 rps** on the same hardware. Splitting 2 GPUs to prefill and 1 to decode yields 10 rps overall = **3.3 rps/GPU, 2.1x** the colocated system, from disaggregation alone.

DistServe therefore runs separate prefill instances and decode instances, streams the KV cache from the former to the latter, and uses a placement-search algorithm (backed by a simulator) to pick per-phase parallelism — intra-op vs. inter-op degree — and replica counts that jointly maximize goodput under the given SLOs. The chosen plans are non-obvious: for OPT-175B on ShareGPT it picked inter-op=3/intra-op=3 for prefill and inter-op=3/intra-op=4 for decode. Placement is bandwidth-aware, keeping a prefill instance and its decode instance's corresponding stage on the same machine so KV transfer rides NVLink. The obvious objection — that shipping the KV cache between GPUs is expensive — is measured away: for OPT-175B, transmission is **under 0.1% of total request latency**, and over 95% of requests see under 30ms of transfer, on a testbed whose cross-node bandwidth is only 25 Gbps.

The paper is also the sharpest published critique of [[chunked-prefill]]. It characterizes chunking as "trading TTFT for TPOT" without eliminating interference, and gives the reason chunked prefill cannot be tuned out of trouble: small chunks leave the prefill competing with decodes and unable to use the GPU alone, large chunks leave no room to piggyback decodes, and chunking forces the KV cache of all previous chunks to be reloaded from HBM for every subsequent chunk — **O(N^2)** KV-cache loads for N chunks versus O(N) unchunked, an overhead that grows with context length. Empirically their chunked-prefill baseline, DeepSpeed-MII, is the one that "struggles to meet the TTFT SLO as a sacrifice for better TPOT."

## Key points

| Result | Number |
|---|---|
| Headline | up to **7.4x** more requests, or **12.6x** tighter SLO, vs. state of the art, at >90% SLO attainment |
| Motivating goodput, 13B on one A100 (512 in / 64 out) | colocated **1.6 rps**; prefill-only 5.6 rps; decode-only 10 rps; disaggregated 3.3 rps/GPU = **2.1x** |
| Chatbot (ShareGPT, OPT-13B/66B/175B) vs. vLLM | **2.0x–4.6x** higher request rate |
| Chatbot vs. DeepSpeed-MII (chunked prefill) | **1.6x–7.4x** higher request rate |
| Chatbot, SLO tightness | 1.8x–3.2x more stringent than vLLM; 1.7x–1.8x more stringent than DeepSpeed-MII |
| Code completion (HumanEval, OPT-66B) | **5.7x** rate and 1.4x tighter SLO vs. vLLM; 1.6x rate and 1.4x tighter SLO vs. DeepSpeed-MII |
| Summarization (LongBench, OPT-66B) | **4.3x** rate and **12.6x** tighter SLO vs. vLLM; 1.8x rate and 2.6x tighter SLO vs. DeepSpeed-MII |
| KV cache transmission cost, OPT-175B | **<0.1%** of total latency; >95% of requests under 30 ms |
| Testbed | 4 nodes x 8 NVIDIA SXM A100 80GB (NVLink intra-node), 25 Gbps cross-node |

- Prefill saturates an A100 with a **single** 512-token sequence for a 13B model — past that point, batching more prefill work does not raise efficiency, it just extends the batch's execution time. This is why prefill instances need little batching and decode instances need a lot.
- Which SLO binds depends on the application: code completion is TTFT-bound (both systems get limited by TTFT), summarization is TPOT-bound because its TTFT budget is loose but its inputs are long.
- Failure mode of colocated vLLM in chat: it usually meets TTFT but a large fraction of requests violate TPOT, because prefills keep stalling the decode steps.
- The authors argue disaggregation gets *more* attractive with long context: KV size grows linearly with prompt length while prefill compute grows quadratically, so transfer becomes relatively cheaper, while prefill/decode interference gets worse.
- Concurrent/related disaggregation work acknowledged in the paper: Splitwise, TetriInfer, DejaVu.

## Concepts & entities

- [[chunked-prefill]] — the main alternative the paper argues against; DistServe quantifies its O(N^2) KV reload cost and its TTFT-for-TPOT trade.
- [[kv-cache]] — must be transferred from the prefill GPU to the decode GPU; the paper's central engineering risk and its measured non-issue.
- [[paged-attention]] — orthogonal memory-management technique; DistServe's vLLM baseline uses it and is still interference-limited.
- [[vllm]] — the primary baseline (colocated, continuous batching + paged attention).
- [[prefix-caching]] — not used here; DistServe's prefill instances recompute each prompt, so caching shared prefixes is a complementary axis.
- [[transformer-architecture]] — the compute-bound-prefill / bandwidth-bound-decode asymmetry is a property of autoregressive transformer inference.
- [[long-context-llm]] — long prompts widen the prefill/decode asymmetry, which the paper claims strengthens the case for disaggregation.
- [[llm-inference-roofline-yuan-2024]] — companion reading for the compute-bound vs. memory-bound framing that motivates the split.

## References

- [arXiv:2401.09670](https://arxiv.org/abs/2401.09670)
