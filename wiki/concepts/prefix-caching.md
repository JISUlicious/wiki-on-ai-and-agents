---
title: Prefix Caching
type: concept
created: 2026-07-27
updated: 2026-07-27
sources:
  - sglang-radixattention-zheng-2023.md
  - prompt-cache-gim-2023.md
  - paged-attention-kwon-2023.md
status: complete
importance: high
---

# Prefix Caching

**Prefix caching** reuses the [[kv-cache|KV cache]] computed for text the model has already processed, so repeated prompt content is **never prefilled twice**. It is the highest-leverage prompt-processing optimization available, because reused prefill is not merely cheaper — it is *skipped*.

The setting is ubiquitous: system prompts, tool definitions, [[agent-skills|skill]] preambles, few-shot examples and conversation history are re-sent byte-identical on every turn of an agent loop.

## Two families

| | **Exact-prefix reuse** | **Modular reuse** |
|---|---|---|
| Example | [[sglang-radixattention-zheng-2023\|RadixAttention]] (SGLang) | [[prompt-cache-gim-2023\|Prompt Cache]] |
| Reuses | only a contiguous shared *prefix* | named segments at **arbitrary positions**, any order |
| Accuracy cost | **none** — bit-identical | **measurable** (see below) |
| Mechanism | radix tree over KV, LRU-leaf eviction, ref counting | PML schema assigns fixed position IDs per module |

### RadixAttention — automatic, exact, free

Retains finished requests' KV in a **radix tree** (one page per token) with leaf-first LRU eviction and reference counting, plus a **cache-aware scheduler** that runs longest-shared-prefix-first — proven equivalent to DFS over the batch's radix tree, and therefore hit-rate-optimal.

- **Up to 6.4× throughput, 3.7× lower latency** vs vLLM v0.2.5 (Llama-2-7B, A10G)
- **Hit rates 50–99%** across benchmarks; scheduling reaches **96% of optimal**
- Overhead when nothing is reusable: **0.2 s of 74.3 s (<0.3%)** — cheap enough to leave on by default
- Production Chatbot Arena over a month: **52.4%** hit rate (LLaVA-NeXT-34B), **74.1%** (Vicuna-33B), **1.7× average first-token latency reduction**

> [!note] It is a prefill optimization, and only that
> Multi-turn chat with **long outputs shows almost no speedup** — once you are generating, prefix reuse has nothing left to give. Judge it on TTFT, not tokens/sec.

### Prompt Cache — reuse without contiguity, at a cost

Precomputes KV for named **prompt modules** declared in a schema and splices them in. The trick is that LLMs tolerate *discontinuous* position IDs, so a module can be reused at any position, in any order, interleaved with uncached text — which exact-prefix caching cannot do.

TTFT gains are large: **1.5–3× on GPU** with modules in CPU memory, **5–10×** with modules resident in GPU memory, and **up to 70×** for CPU inference. Concretely, RTX 4090 / Llama2-7B / 3K context: **900 ms → 90 ms**, with decode unchanged at ~32 ms/token.

> [!warning] The accuracy cost is real and task-specific
> Because each module is encoded **in isolation**, cross-module attention is masked. LongBench scores are broadly comparable overall, but **Passage Retrieval regresses clearly** — 7.50 → 4.25 (Llama2-7B), 9.08 → 6.50 (13B). Tasks requiring reasoning *across* the cached segments are exactly where it breaks. "Scaffolding" (co-encoding a module set) buys consistency back for extra memory.

Memory budget: ~0.5 MB/token (Llama-7B), ~2.5 MB/token (Llama-70B) — caching is not free in HBM.

## Origins

The mechanism descends from [[paged-attention|PagedAttention]]'s ref-counted, copy-on-write block sharing, which already showed **1.67× throughput with a 1-shot shared prefix and 3.58× with a 5-shot prefix**, noting the prompt phase *"only needs to execute on the user's task input."*

## Practical notes

- **Cache hit rate is the metric.** Structure prompts so invariant content comes *first* — anything before the first varying byte is cacheable, anything after is not. A timestamp at the top of a system prompt destroys the whole cache.
- Exact-prefix reuse has no accuracy risk, so prefer it unless you specifically need out-of-order module reuse.

## Related

- [[kv-cache]] · [[paged-attention]] · [[vllm]] · [[agent-skills]] (progressive disclosure keeps tokens out of prefill entirely)
- [[prompt-compression]] — the complement: shrink what *can't* be reused.
- [[quantization-performance]] — why prefill responds to different levers than decode.

## References

- [[sglang-radixattention-zheng-2023]] · [[prompt-cache-gim-2023]] · [[paged-attention-kwon-2023]]
- [[methods-to-speed-up-prompt-processing]] — where prefix caching sits among the other prefill levers
