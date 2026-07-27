---
title: "SGLang: Efficient Execution of Structured Language Model Programs"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - sglang-radixattention-zheng-2023.md
arxiv_id: "2312.07104"
year: 2023
authors:
  - Lianmin Zheng
  - Liangsheng Yin
  - Zhiqiang Xie
  - Ying Sheng
  - Ion Stoica
tags:
  - 2023
  - paper
status: complete
importance: high
---

# SGLang: Efficient Execution of Structured Language Model Programs

**Lianmin Zheng, Liangsheng Yin, Zhiqiang Xie, et al.** (UC Berkeley, Stanford, Shanghai Jiao Tong, Texas A&M; equal contribution Lianmin Zheng and Ying Sheng) — [arXiv:2312.07104](https://arxiv.org/abs/2312.07104). No venue in the arXiv metadata; v2 (June 2024) is marked "Preprint. Under review."

## Summary

SGLang is a co-designed frontend language and serving runtime for **LM programs** — applications that issue many interdependent LLM calls with control flow and structured I/O (agents, tree-of-thought, few-shot benchmarks, JSON decoding, RAG pipelines, multi-turn chat). The frontend is a Python-embedded DSL with primitives for generation (`gen`, `select`, `extend`) and parallelism (`fork`, `join`); the runtime exploits the program structure the frontend exposes.

The runtime contribution that matters for prefill is **RadixAttention**, the canonical automatic prefix-caching mechanism. Existing engines discard a request's KV cache when the request finishes; RadixAttention instead **retains it in a radix tree** that maps token sequences to KV tensors stored in a paged, non-contiguous layout (one token per page). Prefix matching, insertion, and eviction are all tree operations, so KV reuse becomes automatic across calls, across program instances, and across users — no manual annotation of what is shareable. Eviction is **LRU on leaves first** (so common ancestors survive until they themselves become leaves), with a reference counter per node so nodes in the running batch cannot be evicted. Cache and running requests share one memory pool rather than a preallocated cache, so under load the system will evict everything in favor of a larger batch.

Because hit rate depends on request order, RadixAttention pairs the tree with a **cache-aware scheduler**: sort the waiting queue by matched prefix length and run longest-shared-prefix-first, which the paper proves (Theorem 3.1) is equivalent to depth-first traversal of the batch's radix tree and therefore optimal for cache hit rate given a cache at least as large as the longest request. FCFS or random ordering causes thrashing. Two other runtime techniques round out the system: a **compressed finite state machine** for constrained decoding (collapses multi-token deterministic paths so several tokens decode in one step) and **API speculative execution** for closed models. RadixAttention is compatible with continuous batching, paged attention, and tensor parallelism.

## Key points

- **Throughput up to 6.4× and latency down to 3.7×** versus vLLM v0.2.5, Guidance v0.1.8 (llama.cpp), and LMQL v0.7.3, on Llama-2-7B across MMLU (5-shot), HellaSwag (20-shot), ReAct agents, generative agents, tree-of-thought, skeleton-of-thought, LLM judge, JSON decoding, multi-turn chat, and a DSPy RAG pipeline. Mostly NVIDIA A10G (24 GB), fp16; the same trend holds for Mixtral-8x7B and Llama-2-70B under tensor parallelism.
- **Cache hit rate is the mechanism.** Across those benchmarks the achieved hit rate ranges **50%–99%**, and cache-aware scheduling reaches **96% of the optimal hit rate on average**. The ablation (partially disabling matched tokens at runtime on tree-of-thought) shows hit rate driving all four metrics monotonically: higher hit rate → larger batch size → higher throughput and lower first-token/total latency.
- **Two distinct wins from the same mechanism.** Reuse cuts prefill FLOPs (lower time-to-first-token) *and* cuts total KV memory by sharing, which admits a larger batch (higher max throughput). The paper is explicit that on **multi-turn chat with long outputs there is almost no speedup** — sharing is low across sessions and decode time dominates — whereas short-output chat benefits a lot. Prefix caching is a prefill optimization, not a decode one.
- **Overhead is negligible when there is nothing to reuse.** On ShareGPT with no reuse opportunity, 100 requests take 74.3 s of which only **0.2 s (<0.3%)** is radix-tree bookkeeping, because tree operations are linear and cheap. The authors conclude RadixAttention can be on by default.
- **No accuracy cost.** Unlike segment-level schemes, RadixAttention only reuses states for *exact shared prefixes*, where KV values are mathematically identical (KV computation depends only on preceding tokens). The evaluation explicitly disables optimizations that would change computed results so all systems produce the same output.
- **Component ablation** — "No Cache", "No Tree Structure" (flat table cache instead of tree), "FCFS Schedule", "Random Schedule", "No Frontend Parallelism", and "No Frontend Hint" each degrade performance relative to full optimization; the tree structure, the scheduler, and the frontend/runtime co-design all carry weight.
- **Production evidence (Chatbot Arena, one month, one worker per model):** **52.4%** RadixAttention hit rate for LLaVA-NeXT-34B and **74.1%** for Vicuna-33B, from common system messages, reused example images, and multi-turn chat history — cutting first-token latency by an average of **1.7×** for Vicuna-33B.
- **Multi-modal**: hashing input images as radix-tree keys reuses image-token KV across questions about the same image — LLaVA-v1.5-7B goes 0.18 → 1.15 image/s and LLaVA-NeXT-34B video 0.02 → 0.10 frame/s versus the authors' HuggingFace implementations (up to ~6×).
- **Other runtime pieces** — compressed FSM raises JSON-decoding throughput **1.6×** (and skipping FSM preprocessing reuse across a batch costs 2.4×); API speculative execution cuts input-token cost roughly **3×** on a three-field GPT-3.5 extraction. Frontend ergonomics: an equivalent OpenAI-API program needs **2.1× as many lines of code**.
- **Distributed** — tensor parallelism needs no extra synchronization (identical tree ops on sharded caches); data parallelism uses a router meta-tree plus per-worker subtrees with weakly consistent eviction, achieving linear scaling and optimal hit rate on MMLU with four workers.

## Concepts & entities

- [[prefix-caching]] — RadixAttention is the canonical automatic implementation: a radix tree of KV state with LRU-leaf eviction plus longest-shared-prefix-first scheduling.
- [[kv-cache]] — reframed from a per-request scratch buffer into a persistent, shared, tree-structured LRU cache.
- [[paged-attention]] — RadixAttention stores KV in a paged, non-contiguous layout (one page per token) and is explicitly compatible with paged attention.
- [[vllm]] — the primary baseline (v0.2.5); the paper notes RadixAttention was later partially integrated into vLLM as an experimental feature.
- [[attention-mechanism]] — reuse is sound because KV state at a position depends only on preceding tokens, so identical prefixes yield identical KV.
- [[in-context-learning]] — few-shot prompts (5-shot MMLU, 20-shot HellaSwag) are the ideal shared prefix; the exemplar block is prefilled once and reused.
- [[chain-of-thought]] — tree-of-thought and skeleton-of-thought workloads generate the branching call structures that create most of the reuse opportunity.
- [[retrieval-augmented-generation]] — the DSPy RAG pipeline benchmark reuses the KV cache of the common context example.
- [[transformer-architecture]] — positional dependence of KV state is why reuse is restricted to true prefixes here.

## References

- [arXiv:2312.07104](https://arxiv.org/abs/2312.07104)
- Code: github.com/sgl-project/sglang
