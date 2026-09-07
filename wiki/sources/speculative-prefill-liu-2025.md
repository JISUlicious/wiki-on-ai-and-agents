---
title: "Speculative Prefill: Turbocharging TTFT with Lightweight and Training-Free Token Importance Estimation"
type: source
created: 2026-09-03
updated: 2026-09-03
sources: []
arxiv_id: "2502.02789"
venue: ICML 2025
year: 2025
authors:
  - Jingyu Liu
  - Beidi Chen
  - Ce Zhang
tags:
  - 2025
  - paper
status: complete
importance: high
---

# Speculative Prefill: Turbocharging TTFT with Lightweight and Training-Free Token Importance Estimation

**Jingyu Liu (UChicago), Beidi Chen (CMU), Ce Zhang (UChicago)** — [arXiv:2502.02789](https://arxiv.org/abs/2502.02789), cs.CL. v1 5 Feb 2025, v2 19 May 2025. Published at **ICML 2025**. Code: `speculative_prefill` (a monkey-patch on top of [[vllm]] 0.6.3.post1).

> [!note] Speculative prefill is not speculative decoding
> [[speculative-decoding]] uses a draft model to *propose output tokens* that the target then verifies in one batched forward — it accelerates the memory-bound **decode** phase and does nothing for prefill. **SpecPrefill** uses a draft model to *estimate which input tokens matter*, so that the target only prefills a subset of the prompt — it accelerates the compute-bound **prefill** phase (TTFT) and does nothing for decode. Same word, opposite phase. The paper's own Section 3.4 notes the two can share a single draft model (the draft's full-prompt forward, which SpecPrefill needs anyway, produces the draft KV that speculative decoding would need), but they are separate mechanisms.

## Summary

The prefill phase is compute-bound, and for the very common regime of large batches of short-to-medium prompts the bottleneck is the **MLP**, not self-attention — so sparse-attention kernels ([[flash-attention]] variants, MInference, FlexPrefill, DuoAttention) do not help there. The authors' key empirical claim is that an instruction-tuned LLM retains most of its quality when given only a *carefully chosen subset* of prompt tokens, zero-shot, with no fine-tuning. **SpecPrefill** exploits this: a cheap **speculator** (Llama-3.1-8B-Instruct) runs over the full prompt, its attention scores are aggregated into a per-token importance, the top chunks are kept, and only those tokens — carrying their **original position IDs** — are sent to the base model (Llama-3.1-70B-Instruct or Llama-3.1-405B-Instruct-FP8). Total FLOPs drop roughly in proportion to the fraction of tokens dropped, for attention, MLP *and* tensor-parallel all-reduce alike.

Headline results, all verified in the body: on one node of 8×H200 with TP=8, the 405B model reaches **7.66× faster TTFT** at a 10% keep rate (theoretical bound 7.72×, so the implementation is near-optimal), which translates into **up to 7× higher maximum end-to-end QPS** on a LongBench Multi-Doc QA workload while keeping >95% of accuracy. On LongBench with the 70B model, SpecPrefill at 10% keep scores 52.74 vs 53.55 for the dense baseline (98.5%) and 99.7% of MInference's average; at 30% keep it *exceeds* the baseline (53.92). On RULER at 10% keep it preserves retrieval, multi-hop tracking and QA out to 128K, but **aggregation tasks (CWE/FWE) collapse** because every token in the prompt carries information. Versus MInference it delivers **2.54×-6.54×** faster TTFT at 99.5% of its quality for large batches with prompts under 128K.

The important structural fact for this wiki: the base model **never computes KV for dropped tokens**. They are absent for the entire generation, and the paper's Limitation section says multi-turn conversation "can potentially fail" without explicit context recomputation.

## Key points

**Mechanism (Algorithm 1)** — enough to implement:

1. **Speculator forward.** Run the small model over the full prompt of length M. With look-ahead N > 0, greedily decode N more tokens from the speculator (the paper uses **N = 8**; gains beyond 16 are minimal). Store the *queries* of the last prompt token and each decoded token (`store_q=True`). If N = 0 the speculator needs no KV cache at all, which saves memory. Under tensor parallelism, gather Q/K across ranks before scoring.
2. **Raw importance.** For every layer and head, compute the attention of each look-ahead query against every context key: a_ij = Softmax(Q_{M+j} K^T)_i for 0 ≤ i < M, 0 ≤ j < N. Using several look-ahead tokens instead of only the last prompt token is the paper's mitigation for [[attention-sink]] and proximity (recency) bias; it matters most on short prompts.
3. **Aggregate.** From the tensor of shape [N, L, S, H] take the **max over heads H and layers L** (so salient tokens stand out) and the **mean over the N look-ahead tokens**. All layers of the speculator are used; no specific layer is selected.
4. **Denoise and select.** Apply a 1D average pool over the sequence to smooth scores, split the context into contiguous chunks, average within each chunk, and keep the **top-K chunks**. The keep rate is the fraction of chunks retained; it is a fixed per-task hyperparameter in all experiments (adaptive selection is left as future work).
5. **Restore position IDs.** The kept tokens are sent to the base model with their **original, non-contiguous position IDs**, and the first decode position is explicitly set to the original context length M. Example from the paper, ten prompt tokens and three decode tokens:
   - original: `[0,1,2,3,4,5,6,7,8,9]`
   - speculated: `[0,1,3,6,7]`
   - decoding: `[0,1,3,6,7, 10,11,12, ...]`
   The authors call this "crucially essential", especially for position-sensitive retrieval and counting tasks. In RoPE terms this is **positional sparse prefill**: the target's rotary embedding sees the true positions, not a re-indexed contiguous sequence.
6. **Base forward.** The base model prefills the subset with those position IDs, then decodes normally. Its KV cache contains only the kept tokens.

Three variants are evaluated: plain **SpecPrefill** (raw last-token attention, no smoothing), **SpecPrefill Full** (all techniques, no look-ahead), **SpecPrefill Full LAH** (all techniques + 8-step look-ahead).

**Setup.** Speculator Llama-3.1-8B-Instruct (BF16) for both targets; targets Llama-3.1-70B-Instruct (BF16) and Llama-3.1-405B-Instruct-FP8 (neuralmagic). TP=8 for both speculator and base on 8×H100 or 8×H200. vLLM with `enforce_eager=True` and `enable_chunked_prefill=False`. Theoretical FLOP ratio speculator/base: **14.24%** for 70B, **2.96%** for 405B (Table 6). Same family, same tokenizer throughout — cross-family drafting is not tested.

**LongBench, Llama-3.1-70B-Instruct (Table 1)** — best-in-class among compression baselines; `*` marks methods that need the context/question split and whose true compression differs from nominal:

| Method | Rate | Single-Doc QA | Multi-Doc QA | Sum | Few-shot | Code | Synthetic | Avg |
|---|---|---|---|---|---|---|---|---|
| Baseline | — | 50.57 | 53.11 | 25.84 | 66.93 | 52.33 | 72.50 | 53.55 |
| RAG* | 10.38% | 32.32 | 41.17 | 18.86 | 45.40 | 44.76 | 30.42 | 35.49 |
| RAG* | 82.22% | 43.25 | 48.16 | 23.56 | 51.44 | 45.92 | 53.04 | 44.23 |
| LLMLingua* | ~10% | 26.50 | 32.94 | 20.95 | 37.40 | 45.00 | 16.33 | 29.85 |
| LLMLingua* | ~50% | 43.64 | 50.67 | 24.77 | 50.96 | 49.05 | 60.33 | 46.57 |
| LLMLingua* | ~90% | 45.94 | 53.91 | 25.87 | 60.46 | 54.06 | 72.00 | 52.04 |
| MInference | N/A | 50.46 | 53.23 | 25.83 | 66.36 | 52.48 | 69.00 | 52.89 |
| SpecPrefill | **10%** | 47.64 | 52.96 | 21.74 | 64.52 | **63.33** | 66.25 | **52.74** |
| SpecPrefill | 30% | 49.47 | 53.39 | 24.41 | 65.83 | 62.62 | 67.83 | **53.92** |
| SpecPrefill | 50% | 50.18 | 52.56 | 25.10 | 65.60 | 59.91 | 68.17 | 53.59 |
| SpecPrefill | 70% | 50.06 | 52.44 | 25.51 | 65.77 | 58.08 | 68.67 | 53.42 |
| SpecPrefill | 90% | 50.26 | 53.25 | 25.65 | 66.35 | 53.47 | 70.67 | 53.27 |

At 10% keep the average is 98.5% of baseline; summarization loses ~16% relative and synthetic ~9%, while **code completion improves by 11 points** (52.33 → 63.33) — the paper's "noisy query" category where dropping distracting tokens helps. [[llmlingua-jiang-2023]] at ~10% is 29.85 vs SpecPrefill's 52.74 despite being given the context/question split. Figure 2 (405B) shows the same pattern with a smaller gap: the bigger the base model, the better it adapts to the subset.

**RULER, Llama-3.1-70B-Instruct, 10% keep (Table 2):**

| Length | Retrieval (NIAH) | Multi-hop (VT) | QA | Aggregation (CWE/FWE) | Avg w/o Agg |
|---|---|---|---|---|---|
| 4K | 100.0 → 99.7 | 100.0 → **89.6** | 76.9 → 75.2 | 99.7 → **77.9** | 92.3 → 88.2 |
| 8K | 99.9 → 99.6 | 100.0 → 100.0 | 74.7 → 75.6 | 98.0 → 79.7 | 91.5 → 91.7 |
| 16K | 99.8 → 99.5 | 100.0 → 99.1 | 72.0 → 75.3 | 97.8 → 78.5 | 90.6 → 91.3 |
| 32K | 99.6 → 99.7 | 100.0 → 100.0 | 69.8 → 72.6 | 96.9 → 70.0 | 89.8 → 90.8 |
| 64K | 98.5 → 99.5 | 99.9 → 99.8 | 65.1 → 71.9 | 65.6 → 54.9 | 87.9 → 90.4 |
| 128K | 76.5 → **85.8** | 56.1 → 55.6 | 48.2 → 55.3 | 41.3 → 48.3 | 60.3 → 65.6 |

(dense → SpecPrefill.) Retrieval and QA hold or improve at one-tenth of the tokens; the 4K multi-hop dip is because 10% of 4K is only ~400 tokens. Aggregation — "which words are most common in this text" — is the failure mode: every token is evidence, and no importance estimator can drop 90% of them.

**Standard short tasks (Figure 6, 70B).** Generative MMLU, IFEval, GSM8K 8-shot, HumanEval, MBPP, ARC-Challenge, GPQA 8-shot. Numbers are only given graphically; the text says short prompts are "information dense" so low keep rates hurt, but MBPP and GPQA improve at some keep rates and the average can match or beat the baseline "when choosing the right token keep rate". Treat this as: SpecPrefill is a long/medium-context tool, not a general default.

**Efficiency (8×H200, TP=8):**

| Measurement | Result |
|---|---|
| TTFT speedup, 405B-FP8, 10% keep | **7.66×** (Figure 3); theoretical bound 7.72× at 32K tokens, batch 16 |
| Stated TTFT reduction | up to **87%** (vs SwiftKV's 50%) |
| Max end-to-end QPS, 405B, LongBench Multi-Doc QA | **up to 7×** at >95% accuracy (Figure 1) |
| Speculation overhead as share of total FLOPs | higher at low keep rates; 8B is 2.96% of 405B FLOPs |
| vs MInference, 70B, 8×H100, batch large, ≤128K | **2.54×-6.54×** faster TTFT at 99.5% of its LongBench quality (Figure 5) |

Maximum QPS under a fixed timeout scales as O(1/TTFT), which is why a TTFT cut converts almost one-for-one into serving capacity. At low keep rates the 405B model with SpecPrefill serves more QPS than the dense 70B. MInference only wins back when sequence-length/batch-size is very large (ultra-long single prompts), because its pattern-search overhead is amortized only there.

**Where it breaks.** (1) Information-dense prompts (short tasks, aggregation, counting). (2) No per-token logits for dropped tokens, so only generative evaluation is possible — no perplexity or classification over the full prompt. (3) **Multi-turn**: the base model's KV cache lacks the dropped tokens, so a follow-up question about dropped content cannot be answered unless the context is recomputed; the authors frame "compute and store all KV during prefill" vs "drop" as an open trade-off and call for dynamic KV recomputation. (4) Fixed keep rate; a compressibility-aware budget is future work. (5) Chunk-level selection means the speculator must be a reasonable proxy for the target's attention — the paper only demonstrates this within the Llama-3.1 family.

**Answers to the wiki's standing questions.**
- *Positions:* kept tokens keep their **original position IDs** in the target's RoPE; decode starts at position M (original context length). Not re-indexed.
- *Cross-family draft:* **not tested**; speculator and target are both Llama-3.1 and the stated insight is transfer "within the same family". See [[cross-family-speculative-prefill-upasani-2026]] for the follow-up that addresses this.
- *Dropped-token KV:* **does not exist**. The target computes KV only for kept tokens; dropped tokens are gone for the whole generation and across turns unless recomputed.
- *Aggressive keep ratios:* at 10% keep, LongBench 70B retains 98.5% of the baseline average and RULER retrieval/QA is flat or better; summarization and aggregation are the casualties. 20% is not reported separately (the grid is 10/30/50/70/90%).

## Related

- [[speculative-prefill]] — the technique this paper introduces (no concept page yet).
- [[speculative-decoding]] — the decode-phase cousin; explicitly *not* what this is, though the same draft model can serve both.
- [[methods-to-speed-up-prompt-processing]] — the wiki's prefill-speedup synthesis; SpecPrefill is the token-dropping entry that also shrinks MLP cost.
- [[prompt-compression]] — SpecPrefill as a token-level, model-internal compression method; compare [[llmlingua-jiang-2023]] and [[llmlingua-2-pan-2024]], which it beats by a wide margin at matched rates.
- [[kv-cache]] — the base model's cache holds only kept tokens; dropped tokens have no KV.
- [[positional-encoding]] / [[rope-scaling]] — original position IDs are preserved (positional sparse prefill).
- [[attention-mechanism]] / [[attention-sink]] — token importance is aggregated attention; look-ahead counteracts sink and recency bias.
- [[long-context-llm]] — LongBench and RULER are the main evaluations.
- [[llm-inference-roofline-yuan-2024]] — prefill is compute-bound; the MLP dominates for large batch × short sequence.
- [[chunked-prefill]] / [[sarathi-chunked-prefill-agrawal-2023]] — orthogonal scheduling technique; disabled in these experiments.
- [[flash-attention]] / [[minference-dynamic-sparse-attention-jiang-2024]] / [[minimax-sparse-attention-lai-2026]] — attention-only accelerations that do not touch the MLP; MInference is the sparse-attention baseline here.
- [[discovering-the-gems-in-early-layers-shi-2024]] (GemFilter) / [[lazyllm-dynamic-token-pruning-fu-2024]] — prior token-dropping methods that use the *target model's own* early-layer attention; SpecPrefill replaces that extra target pass with a cheaper separate speculator.
- [[retrieval-augmented-generation]] — sentence-level RAG is a baseline; SpecPrefill is framed as token-granular RAG with the draft's attention as the relevance metric.
- [[cross-model-kv-cache-transfer-heo-2026]] — a different route to avoiding re-prefill (reuse a sibling model's cache rather than dropping tokens).
- [[llama]] / [[vllm]] / [[cmu]] — models, serving engine, affiliation.

## References

- [arXiv:2502.02789](https://arxiv.org/abs/2502.02789) — v2, 19 May 2025; ICML 2025.
- Follow-ups: [[cross-family-speculative-prefill-upasani-2026]] (cross-family drafts), [[draft-based-approximate-inference-galim-2025]] (generalizes draft-based importance to KV dropping and sparse attention).
