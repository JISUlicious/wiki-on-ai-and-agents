---
title: Long-Context LLM
type: concept
created: 2026-05-16
updated: 2026-10-06
sources:
  - em-llm-fountas-2024.md
  - titans-behrouz-2025.md
status: draft
importance: medium
tags:
  - 2024
---

# Long-Context LLM

LLMs with very large context windows — 200K, 1M, or 10M+ tokens — that can fit large bodies of text directly in the prompt rather than relying on external retrieval. By 2024–2026 the major frontier models (Claude 3, Gemini 1.5, GPT-4-Turbo) all advertise long context as a headline feature, and "just put everything in the context" became a credible alternative to [[retrieval-augmented-generation|RAG]] for many workloads.

> [!info] Why memory still matters
> Long context is **uneconomic at scale**: vanilla attention is O(n²) in sequence length, and the [[kv-cache|KV-cache]] grows linearly with tokens. So even at 1M+ tokens, memory-management systems remain valuable. [[em-llm]] and [[titans]] both treat long context as a **memory representation problem**, not a substitute for [[memory-management|memory management]].

## Where it appears

- [[em-llm]] — extends effective context by structuring the [[kv-cache|KV-cache]] into retrievable episodic events.
- [[titans]] — adds learned [[long-term-memory|long-term memory]] modules alongside attention to push effective context further.
- Frontier-model context windows: Claude 3 (200K), Gemini 1.5 (1M–10M), GPT-4-Turbo (128K).

## References

- [[em-llm-fountas-2024]]
- [[titans-behrouz-2025]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[flashmemory-lookahead-sparse-attention-wang-2026]] — FlashMemory/LSA: lookahead sparse attention for ultra-long context
- [[can-lms-actually-retrieve-in-context-gollapudi-2026]] — million-token in-context retrieval fails at *readout*, not ranking — gold attention share collapses 0.91 → 0.01
- [[recontext-recursive-evidence-replay-zhao-2026]] — training-free long-context harness using model-internal attention as the relevance signal (+24.6% relative)
- [[flash-attention]] — makes long context feasible: attention memory goes quadratic → linear
- [[prompt-compression]] — fewer tokens entering prefill at all; task-agnostic compression stays cacheable
- [[chunked-prefill]] — how long prompts get scheduled against concurrent decodes
- [[linear-attention]] — subquadratic prefill; the advantage only appears past ~128k (2.9× at 1M)
- [[frontier-lms-struggle-to-copy-wen-2026]] — frontier models fail to copy text well inside their window; traced to 1D positional encodings, fixed by a 2D view
- [[pro-long-programmatic-memory-fox-2026]] — programmatic memory over a verbatim log as an alternative to summarization
- [[attention-sink]] — and the sign inversion in that same sweep: sinks correlate *positively* with long-context performance (R²=0.38), against the prior reading of them as a pathology
- [[cracks-in-the-foundation-long-context-extension-bertsch-2026]] — four minor architectural choices — normalization, GQA, pretraining context length, SWA — compound to **−47% relative** long-context performance, invisible in loss (end-of-extension loss explains R²=0.06)
- [[lost-in-compaction-side-constraint-loss-wang-2026]] — constraint retention rises *monotonically* with injection position, diverging from the U-shaped lost-in-the-middle pattern
- [[rope-scaling]] — how an advertised window gets extended past the trained one — and why the extension needs its own evaluation rather than inheriting native-length numbers
- [[qwen3.8-flash-next]] — a 2026 worked example: 256K native, 1M via YaRN 4×, with no published number measured on the extended path
- [[sparse-prefill]] — the attention-share rule: sparse attention is Amdahl-bounded below ~64K and the only lever past 300K; token dropping is the reverse
- [[sparse-attention]] — MInference reaches 10× end-to-end at 1M on one A100 — and ×0.47, slower than dense, at LongBench lengths
- [[trained-sparse-attention]] — NSA, MoBA, DSA — sparsity in the training loop
- [[parser-read-in-parallel-reason-in-depth-li-2026]] — read in parallel, reason in depth: 83–86% at every length up to 896K where 4B full-context reading falls to 31%
- [[residual-vector-reconstruction-long-context-recall-ryu-2026]] — recall beyond the context window at near-constant memory: facts extracted up front are encoded as feed-forward residual vectors, then regenerated on demand. 50–80% single-fact accuracy at 2M tokens, but from 3–5 questions per model, and routing fails on paraphrased questions
- [[hysparse2-hybrid-sparse-attention-two-level-kv-sharing-wei-2026]] — HySparse2 (Xiaomi): YOCO-style KV bridging plus token-level sparse KV reuse; prefill exits at half depth

## Lineage sources (added 2026-10-06)

- [[lost-in-the-middle-liu-2023]] — U-shaped use of long context (Jul 2023; TACL 2024)
- [[gemini-1-5-google-2024]] — Gemini 1.5: 128K standard, 1M preview, 10M in research (Feb 2024)
- [[context-rot]] — degradation as context grows
- [[context-engineering]] — the practice that responds to it
