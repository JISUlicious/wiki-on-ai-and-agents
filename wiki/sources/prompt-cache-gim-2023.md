---
title: "Prompt Cache: Modular Attention Reuse for Low-Latency Inference"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - prompt-cache-gim-2023.md
arxiv_id: "2311.04934"
year: 2023
venue: MLSys 2024
authors:
  - In Gim
  - Guojun Chen
  - Seung-seob Lee
  - Nikhil Sarda
  - Anurag Khandelwal
  - Lin Zhong
tags:
  - 2023
  - paper
status: complete
importance: high
---

# Prompt Cache: Modular Attention Reuse for Low-Latency Inference

**In Gim, Guojun Chen, et al.** (Yale University; Nikhil Sarda at Google, Mountain View) — [arXiv:2311.04934](https://arxiv.org/abs/2311.04934), Proceedings of the 5th MLSys Conference (2024).

## Summary

Prompt Cache attacks the **prefill** cost of LLM inference by precomputing the key-value attention states of *frequently reused text segments* and splicing them into new prompts instead of recomputing them. The observation is that real prompts are largely made of repeated material — system messages, prompt templates, and documents drawn from a fixed pool — so the attention states for those segments can be computed once at "schema load" time and reused across many later requests. Reuse happens at the granularity of a **prompt module**: a named, self-contained span declared in a **Prompt Markup Language (PML)** schema, which a user's prompt then "imports" by tag (`<miami/>`).

The critical difference from ordinary KV-cache reuse is that **Prompt Cache does not require the reused text to be a contiguous prefix**. Ordinary KV Cache reuses states within a single request; the paged-attention style of prefix sharing reuses states only when two prompts share an identical leading prefix. Prompt Cache reuses arbitrary segments that may appear at different positions, in different orders, and interleaved with uncached text. Two ideas make this legal: (1) the PML schema assigns each module a fixed range of **position IDs** derived from its slot in the schema, so a module's states are always positionally consistent; and (2) the authors' empirical finding that LLMs tolerate **discontinuous position IDs** — as long as relative order within a segment is preserved, quality is not affected, so unimported modules simply leave gaps in the position space. At serve time the imported modules' KV tensors are concatenated (pure memory copy), the uncached remainder is computed, and the result replaces the prefill operation entirely.

The tradeoff is an **approximation**: because each module is encoded in isolation, attention is effectively masked to the module's own span — cross-module attention never happens during encoding. The authors argue this mirrors locally-masked attention (Longformer-style) and can even help by filtering irrelevant context, but it requires modules to be semantically self-contained. For applications that cannot accept it, **scaffolding** lets a set of modules be co-encoded to share an attention span, trading memory for output consistency. The prototype is ~3K lines of Python on HuggingFace `transformers`, needing roughly 20 lines of change per model to support discontinuous position IDs (RoPE and ALiBi both need a position-ID-indexed lookup table; learned-embedding models like BERT/GPT-2 need none).

## Key points

- **TTFT reduction, GPU** — 1.5×–3× when prompt modules live in CPU memory (host-to-device copy on the critical path), and **5×–10× when modules live in GPU memory**. Measured on RTX 4090, A40, and A100 with Llama2 7B over LongBench (~5K-token prompts).
- **TTFT reduction, CPU** — up to **70× on an Intel i9-13900K** (DDR5-5600) and **20× on an AMD Ryzen 9 7950X** (DDR4-3600). CPU inference benefits far more because CPU attention compute is slow relative to memcpy. The abstract summarizes the whole range as "8× for GPU-based inference to 60× for CPU-based inference."
- **Concrete latency point** — RTX 4090 / Llama2 7B / 3K context: TTFT **900 ms → 90 ms**. Time-to-subsequent-token is unchanged at ~32 ms/token, so the win is strictly on prefill; end-to-end benefit shrinks as generation length grows. Code-generation use case (CodeLlama 7B, each source file a module): **4× GPU TTFT improvement with identical output**.
- **Accuracy cost is small but not zero.** LongBench accuracy with Prompt Cache is comparable to the KV-Cache baseline across Llama2 7B/13B, MPT 7B, and Falcon 7B under deterministic (greedy) decoding — some tasks improve, some regress. The clearest regression is **Passage Retrieval**: Llama2 7B 7.50 → 4.25 and Llama2 13B 9.08 → 6.50. Summarization (GovReport/MultiNews Rouge-L) and several QA scores are flat or slightly better. This is the expected signature of the cross-module attention mask: tasks that need to relate one document to another lose the most.
- **Why the advantage grows** — attention compute is quadratic in sequence length (prefill FLOPs ≈ 6nd² + 4n²d) while Prompt Cache's memcpy overhead is linear, so the gap widens quadratically with context length, and also with model size: going 7B → 13B at 3K tokens added 220 ms to the KV-Cache baseline but only 30 ms to Prompt Cache. At 5K tokens, memcpy latency is 3.79 ms host-to-host, 5.34 ms host-to-device, 0.23 ms device-to-device.
- **Memory overhead** — per cached token at FP16: BERT 0.03 MB, Falcon 1B 0.18 MB, Llama 7B 0.50 MB, Llama 13B 0.78 MB, MPT 30B 1.31 MB, Falcon 40B 1.87 MB, Llama 70B 2.5 MB, Falcon 180B 4.53 MB. A 1K-token module costs ~180 MB on Falcon 1B but ~2.5 GB on Llama 70B, which pushes large models toward CPU-memory storage.
- **PML features** — parameters (`<param name len>`, filled at runtime with argument tokens taking over the reserved `<unk>` position IDs, so templates stay cacheable), **unions** (mutually exclusive modules sharing a starting position ID, conserving position space), nested modules, and LLM-specific chat-template tags. Python prompt programs can be auto-compiled to PML (`if` → `<module>`, `if/else` → `<union>`, calls → nested modules).
- **Composes with, not replaces, other work** — explicitly orthogonal to multi-GPU inference, fused attention kernels, and paged attention; the authors note paged attention can dedupe shared modules across a batch to raise throughput, and flag in-context RAG as a natural fit where the retriever is effectively a database of prompt modules.

## Concepts & entities

- [[kv-cache]] — the baseline this extends; Prompt Cache generalizes intra-request KV reuse to inter-request, non-prefix segment reuse.
- [[prefix-caching]] — the neighboring technique Prompt Cache explicitly goes beyond, since reused modules need not form a contiguous prefix.
- [[attention-mechanism]] — the module-local encoding is equivalent to masking attention to each module's own span, which is the paper's central approximation.
- [[transformer-architecture]] — reuse depends on positional encoding; the paper patches RoPE and ALiBi with position-ID lookup tables to allow discontinuous IDs.
- [[paged-attention]] — cited as prior art for simple identical-prefix sharing and as a complement for deduplicating shared modules within a batch.
- [[flash-attention]] — named as an orthogonal kernel-level optimization that Prompt Cache stacks on top of.
- [[long-context-llm]] — evaluated on LongBench (4K–10K contexts); the payoff scales quadratically with prompt length.
- [[retrieval-augmented-generation]] — proposed follow-on: treat the retrieval corpus as a pool of precomputed prompt modules.

## References

- [arXiv:2311.04934](https://arxiv.org/abs/2311.04934)
- Code and evaluation data: github.com/yale-sys/prompt-cache
