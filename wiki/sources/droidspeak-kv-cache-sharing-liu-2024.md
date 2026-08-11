---
title: "DroidSpeak: KV Cache Sharing for Cross-LLM Communication and Multi-LLM Serving"
type: source
created: 2026-08-11
updated: 2026-08-11
sources:
  - droidspeak-kv-cache-sharing-liu-2024.md
arxiv_id: "2411.02820"
year: 2024
authors:
  - Yuhan Liu
  - Yuyang Huang
  - Jiayi Yao
  - Shaoting Feng
  - Junchen Jiang
tags:
  - 2024
  - paper
status: complete
importance: medium
---

# DroidSpeak: KV Cache Sharing for Cross-LLM Communication and Multi-LLM Serving

**Yuhan Liu, Yuyang Huang, Jiayi Yao, Shaoting Feng et al.** (University of Chicago; Shan Lu, Madan Musuvathi and Esha Choukse at Microsoft) — [arXiv:2411.02820](https://arxiv.org/abs/2411.02820), cs.MA, submitted 5 Nov 2024 (v4, 14 Jul 2025). No venue in the arXiv comments.

## Summary

Compound AI systems run several LLMs — agents, LoRA/fine-tuned variants, per-user personalized assistants — that repeatedly process the *same context prefix*. [[prefix-caching]] only helps when the same model reuses its own cache, so every handoff between models pays a full prefill. DroidSpeak asks whether one model can consume another's [[kv-cache]], under the explicit assumption that **the two LLMs have different weights but the same architecture** (typically fine-tuned siblings of one foundation model).

Naive full reuse fails: feeding the sender's whole KV cache to the receiver costs **over 50% of the accuracy points on HotpotQA for most model pairs**. But the damage is not spread evenly. Reusing one layer at a time (recomputing everything else) shows only a small subset of layers are sensitive — **11% of layers are critical on average across all pairs** — and, crucially, which layers are critical is stable across inputs (only critical layers show wide per-input variance; non-critical layers show almost none). That stability is what makes an *offline* decision viable.

DroidSpeak therefore **selectively recomputes a few contiguous layer groups and reuses the sender's KV for the rest**, choosing the group by offline profiling of a quality-vs-latency Pareto frontier (profiled on 50 HotpotQA contexts, then applied unchanged to other datasets). A complication: at each transition from reuse to recompute, the receiver needs the sender's *embeddings* at that layer — the **E cache** — which is up to 2× the KV cache size for Mistral-7B/Llama-3-8B families and up to 4× for Llama-3.1-70B (whose KV is already shrunk by [[grouped-query-attention]]). Each transition also injects fresh error that propagates onward, so recomputing *all* critical layers scattered across the depth is worse than recomputing one contiguous group. Pipelining the layer-wise recomputation against the network load of the reused cache hides much of the remaining transfer cost.

## Key points

- **Mechanism**: reuse the sender's KV cache for most layers; **recompute a contiguous group of "critical" layers**; ship the E cache (layer-input embeddings) at each transition point. Figure 9's running example recomputes just **layers 4-5**.
- **Critical layers**: ~**11% of layers** on average are sensitive to KV reuse. Sensitivity is a property of the model pair, not the input — enabling one-time offline profiling.
- **Why contiguous**: each reuse→recompute transition loads a foreign E cache, and that error propagates to later critical layers. More transition points means higher output error, so scattered critical layers are grouped rather than individually recomputed.
- **Prefill latency**: **1.7-3.1× faster**, **2.1× average**, across 3 datasets and 8 model pairs, without compromising accuracy. Code agentic workload: **2.7× better TTFT**.
- **Throughput**: up to **4×** in online serving (2-4× higher sustainable load before the TTFT knee). Gains come from cutting TTFT; TBT and E2E improve indirectly by avoiding queuing collapse.
- **Naive baseline**: full KV reuse loses **>50% of accuracy points on HotpotQA** for most pairs — the quality floor that motivates selective recomputation.
- **Setup**: 8 model pairs (fine-tuned Mistral-7B, Mistral-24B, Llama-3.1-8B, Llama-3-8B, Phi-3.5-mini-instruct, Llama-3-70B, Llama-3.1-70B — the 70Bs AWQ 4-bit quantized), 6 LongBench-style datasets (hotpotQA, 2wikimQA, multifieldQA_en, multi_news, lcc, repobench-p), metrics F1 / Rouge-L / code-similarity. 2× A100 VMs (8×80GB each) over InfiniBand, ~3K lines on PyTorch 2.0 + LMCache.
- **Robustness**: the Pareto frontier profiled on one dataset transfers to other datasets and model types.

## How it differs from [[cross-model-kv-cache-transfer-heo-2026]]

Confirmed — the two solve the same problem with opposite instruments:

| | DroidSpeak (2024) | Cross-Model KV Cache Transfer (NVIDIA, 2026) |
|---|---|---|
| Core move | **Recompute** selected layers, reuse the rest verbatim | **Learn a mapping** from source KV to target KV; recompute nothing |
| Model relationship | Same architecture, different weights (fine-tuned siblings) | Different *sizes* in one family; differing layer count and hidden dim, only KV head count and per-head dim matched |
| Cost paid | Partial prefill (~11% of layers) + E-cache transfer | One-time offline ridge fit (~47-87 min, 1-3.4 B mapper params) + a batched matmul at serve time |
| Extra state shipped | KV cache **plus E cache** (up to 2-4× KV size) | KV cache only |
| Fitting | None — offline profiling only picks *which* layers to recompute | Closed-form ridge regression on 500 calibration sequences |
| Reported gain | 1.7-3.1× prefill (2.1× avg), up to 4× throughput | 2.7-25× prefill latency vs. re-prefill |

The 2026 paper cites DroidSpeak directly as prior work that "requires identical architecture", which is what rules it out for the cost-quality cascading case (14B→32B) that motivates the linear mapper. Conversely DroidSpeak needs no per-pair fitting and no mapper storage, and its accuracy claim is stronger (negligible loss vs. 73-98% retention on the good pairs, with sharp failures on two).

## Concepts & entities

- [[kv-cache]] — the shared object; DroidSpeak additionally introduces the E (embedding) cache at transition layers.
- [[prefix-caching]] — the single-model technique DroidSpeak generalizes across models.
- [[cross-model-kv-cache-transfer]] — the problem class; DroidSpeak is the recompute-based point in this design space.
- [[cross-model-kv-cache-transfer-heo-2026]] — the learned-mapping alternative that cites this work.
- [[grouped-query-attention]] — shrinks KV but not E cache, so the E-cache overhead is worst (4×) on GQA-heavy Llama-3.1-70B.
- [[chunked-prefill]] — related prefill-scheduling technique; DroidSpeak instead pipelines partial recompute against cache loading.
- [[quantization]] — the 70B pairs are evaluated as AWQ 4-bit models.
- [[vllm]] — the serving-system context for KV reuse and throughput measurement.

## References

- [arXiv:2411.02820](https://arxiv.org/abs/2411.02820)
