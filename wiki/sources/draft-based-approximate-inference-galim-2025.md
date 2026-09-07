---
title: "Draft-based Approximate Inference for LLMs"
type: source
created: 2026-09-03
updated: 2026-09-03
sources: []
arxiv_id: "2506.08373"
venue: ICLR 2026
year: 2025
authors:
  - Kevin Galim
  - Ethan Ewer
  - Wonjun Kang
  - Minjae Lee
  - Hyung Il Koo
  - Kangwook Lee
tags:
  - 2025
  - paper
status: complete
importance: high
---

# Draft-based Approximate Inference for LLMs

**Kevin Galim\*, Ethan Ewer\*, Wonjun Kang, Minjae Lee, Hyung Il Koo, Kangwook Lee** (FuriosaAI; UW-Madison; Seoul National University; Ajou University; KRAFTON; \* equal contribution) — [arXiv:2506.08373](https://arxiv.org/abs/2506.08373), cs.CL. v1 10 Jun 2025, v3 2 Feb 2026 (this summary follows v3). **Accepted to ICLR 2026.** Code: `furiosa-ai/draft-based-approx-llm`.

> [!note] Speculative prefill is not speculative decoding
> The paper says it itself (Section 3): [[speculative-decoding]] has a draft propose output tokens the target verifies — it "improves hardware utilization ... without changing the output distribution" but "does not reduce total computation or memory usage." This framework instead uses the draft to *predict which input tokens or KV pairs matter* so that the target can skip them: an approximate-inference, prefill/memory-side technique. Same draft model, opposite phase and opposite guarantee (approximate, not exact).

## Summary

Every approximate-inference method — KV-cache dropping (SnapKV, H2O, PyramidKV), sparse prefill (MInference, FlexPrefill), prompt compression (LLMLingua-2, CPC, R2C) — needs an estimate of which input tokens future outputs will attend to. Most estimate it from the *input's own* attention, because the outputs do not exist yet. The authors' framing, **Draft-based Approximate Inference**, is that a cheap draft model can manufacture an *approximate future*: generate a look-ahead, and use attention toward (or from) those draft tokens as the importance signal. This unifies [[speculative-prefill-liu-2025]] (draft attention → prompt compression) and Lookahead Q-Cache/LAQ++ (draft queries → KV dropping), and two theorems bound the importance-score error by the draft's output error (Theorem 1 for KV importance; Theorem 2, via a Restricted-Isometry-Property argument borrowed from compressed sensing, for attention approximation).

Three concrete methods: **SpecKV** — the target prefills the full prompt *plus* the draft's look-ahead tokens, scores each KV pair per head by the max attention it receives from a 32-token window of late prompt queries and the draft-token queries, uses those scores both for Vertical-Slash sparse prefill and to keep only C_max = 256 KV pairs per head after prefill. **SpecPC** — an extension of SpecPrefill that reads the draft's attention directly (skipping the first 8 layers, weighting a 64-query window toward the end, max over layers/heads/queries, avg- then max-pooling to keep neighbourhoods) and passes the top-C_max tokens to the target **with contiguous re-indexed position IDs**. **SpecKV-PC** — cascade: SpecPC to 2,048 tokens, then SpecKV to 256 KV entries.

On LongBench (14 English tasks, 50 examples each) with Qwen2.5-32B and 4-bit Llama-3.1-70B targets: SpecKV at C_max=256 scores **46.06 / 48.80** vs dense **47.78 / 49.99** (SnapKV 42.98 / 47.75, LAQ++ 45.79 / 48.43); SpecPC at C_max=1,024 scores **43.66 / 48.44** vs SpecPrefill **42.70 / 48.37**, CPC 40.91 / 41.97, LLMLingua-2 30.90 / 33.63; and SpecKV-PC-2048 reaches **46.48 / 51.60** — on Llama *above* the dense target. On RULER (4K-64K) SpecKV and SpecPC beat every baseline at every length (Figure 5). Cross-family drafts (Qwen2.5 → Llama-3.1-70B) are fine for SpecKV but **cost SpecPC 2-6 points**, the opposite of what [[cross-family-speculative-prefill-upasani-2026]] reports.

## Key points

**SpecKV (Algorithm 1)** — draft look-ahead for KV dropping and sparse prefill:

1. Draft generates y_draft of length n_lookahead (until EOS for accuracy runs; 64 for latency runs; n_lookahead = 0 reduces exactly to SnapKV).
2. **The target model** forwards [prompt; y_draft]. Per head at each layer, with m = n_in − n_window (n_window = 32): cross-attention from queries of the last-window prompt tokens *and* the draft tokens to keys of the first m prompt tokens.
3. s = MaxReduce over queries → AvgPool1D(k = 7).
4. **Sparse prefill**: top n_vert = 2,048 global key columns plus an n_slash = 2,048 sliding window, in MInference's Vertical-Slash kernel. Accuracy plateaus at 2,048 (Appendix E.8.3); more sparsity occasionally *helps* on some LongBench categories.
5. **KV dropping**: keep top (C_max − n_window) keys by s plus the last n_window keys. All other KV entries are evicted after prefill.

Because the draft supplies the look-ahead, SpecKV never has to hold the target's full cache the way LAQ++ does (LAQ++ drafts *with the target* on a compressed cache, then rescores — so its peak memory equals dense). Importance R² against oracle (target-output) scores rises monotonically with n_lookahead; n_lookahead = 32 (6.25% of a 512-token output) already beats SnapKV (Appendix E.4), and R² tracks downstream score.

**SpecPC (Algorithm 2)** — the SpecPrefill descendant:

1. Run the draft over the prompt with n_lookahead = 1 — i.e. **no extra decode steps**; the attention tensor has n_in + n_lookahead − 1 = n_in query rows. (Longer look-ahead gave only "minor accuracy gains", Appendix E.8.2.) A ∈ R^{n_layer × n_head × n_in × n_in}.
2. Skip the first l_skip = 8 layers (following PyramidKV's observation that attention focuses in deeper layers). Restrict to the last n_window = 64 query rows and the non-window key columns; weight the j-th window query by j / n_window so later prompt tokens count more.
3. s = MaxReduce over layers, heads, and queries. (Liu et al. used mean over look-ahead, max over layers/heads; here max throughout, which the authors found better.)
4. AvgPool1D(k = 64; 32 for Llama) then **MaxPool1D(n_neighbor = 64; 32 for Llama)** — the max-pool is what keeps each selected token's neighbourhood, replacing SpecPrefill's fixed chunking that could split a key-value pair.
5. Select top-C_max tokens ∪ the window tokens. C_max ∈ {1,024, 2,048, 3,072}.
6. The target receives the compressed prompt with **reassigned contiguous position IDs** — the paper notes this lets SpecPC "surpass the model's maximum context length".

Draft and target attention importance correlate strongly (Figure 2b, Qwen2.5-1.5B vs 14B) and the correlation rises with draft size (Figure 2c). SpecPC is deliberately sentence-agnostic so it also works on image tokens; on six MileBench tasks with Qwen2.5-VL-3B → 32B it beats FastV (Appendix E.3).

**SpecKV-PC** — SpecPC compresses to X tokens, then SpecKV drops KV to 256; the draft's look-ahead tokens are reused for both stages. Moderate pre-compression (X = 2,048) *improves* on SpecKV alone because it removes easy-to-spot junk before the target's finer per-head selection; aggressive pre-compression collapses (Table 8, LongBench "All"):

| Method | Qwen2.5-32B | Llama-3-70B (4-bit) |
|---|---|---|
| Dense target | 47.78 | 49.99 |
| SnapKV (256) | 42.98 | 47.75 |
| LAQ++ (256) | 45.79 | 48.43 |
| SpecKV (256) | 46.06 | 48.80 |
| SpecKV-PC-2048 | **46.48** | **51.60** |
| SpecKV-PC-1024 | 45.28 | 49.89 |
| SpecKV-PC-512 | 40.21 | 46.68 |
| SpecPC-256 (prompt only) | 32.00 | 33.50 |

**Main LongBench results (Table 2)** — Qwen2.5-32B with 1.5B (KV) / 0.5B (PC) drafts; Llama-3.1-70B 4-bit with Llama-3.2-3B / 1B drafts:

| Group | Method | Qwen SingleQA | MultiQA | Summ | Few-shot | Code | **All** | Llama SingleQA | MultiQA | Summ | Few-shot | Code | **All** |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Dense | Target | 56.01 | 43.99 | 25.90 | 64.06 | 44.74 | 47.78 | 55.02 | 47.06 | 28.61 | 70.47 | 48.19 | 49.99 |
| KV 256 | H2O | 46.63 | 30.81 | 19.88 | 56.03 | 39.27 | 39.32 | 54.07 | 41.30 | 22.55 | 49.10 | 54.14 | 43.52 |
| | SnapKV | 52.54 | 40.21 | 19.89 | 61.18 | 40.12 | 42.98 | 55.88 | 45.30 | 22.49 | 62.15 | 55.49 | 47.75 |
| | PyramidKV | 50.92 | 37.26 | 18.90 | 63.24 | 40.20 | 43.19 | 55.41 | 45.59 | 22.50 | 59.06 | 49.90 | 46.25 |
| | LAQ++ | 55.15 | 44.14 | 22.24 | 63.25 | 41.19 | 45.79 | 54.90 | 46.48 | 22.83 | 64.31 | 55.10 | 48.43 |
| | SpecKV | 53.48 | 43.77 | 24.02 | 63.79 | 44.80 | 46.06 | 51.80 | 47.23 | 25.53 | 64.02 | 58.75 | 48.80 |
| | SpecKV-PC-2048 | 52.60 | 44.52 | 24.11 | 63.38 | 48.45 | 46.48 | 61.42 | 47.15 | 26.51 | 66.94 | 58.19 | 51.60 |
| PC 1024 | LLMLingua-2 | 33.83 | 26.39 | 22.85 | 32.46 | 43.01 | 30.90 | 37.95 | 28.20 | 23.35 | 42.37 | 37.63 | 33.63 |
| | CPC | 45.60 | 40.62 | 23.09 | 60.08 | 32.31 | 40.91 | 45.14 | 39.41 | 24.86 | 61.40 | 37.58 | 41.97 |
| | R2C | 50.49 | 40.37 | 23.26 | 53.45 | 34.11 | 39.88 | 48.93 | 42.01 | 25.38 | 58.91 | 40.19 | 43.29 |
| | SpecPrefill | 45.94 | 39.32 | 23.16 | 62.04 | 43.17 | 42.70 | 54.62 | 46.43 | 25.63 | 64.80 | 44.92 | 48.37 |
| | SpecPC | 51.23 | 41.40 | 23.37 | 62.26 | 38.23 | 43.66 | 56.84 | 44.48 | 25.91 | 67.37 | 47.15 | 48.44 |

Context for the budgets: the drafts alone score 30.37 (Qwen-0.5B) / 39.06 (1.5B) and 35.27 (Llama-1B) / 44.89 (3B), so every Spec method far exceeds its own draft. C_max=1,024 tokens against LongBench prompts of ~1.2K-18K words is a keep ratio anywhere from roughly 5% to 60% depending on task; on RULER at 64K it is 1.6%, and a 256-entry KV budget is 0.4%. SpecPrefill (re-implemented here with n_lookahead = 8, n_window = 1, chunk 32, mean-max) is the strongest prompt-compression baseline and within 0.1-1.0 of SpecPC on average — SpecPC's edge is concentrated in single-doc QA; SpecPrefill keeps more on code.

**Efficiency (Section 5.3, one H200 141 GB, Qwen2.5-32B target, 1.5B/0.5B drafts, latency = time to generate 64 tokens including draft stages).** SpecKV is faster than SnapKV and LAQ++ thanks to sparse prefill and a cheap draft; SpecKV-PC cuts latency **75% vs LAQ++ at 64K** and saves ~25 GB peak memory (LAQ++ must keep the dense cache). SpecPC has the lowest latency of all prompt-compression methods — faster than SpecPrefill because n_lookahead = 1 vs 8, and free of CPC/R2C's CPU preprocessing. Absolute TTFT/QPS figures are given only graphically (Figures 6, 16, 17); no numeric TTFT speedup is stated. Appendix E.7's separate sweep (3B draft, n_lookahead = n_out) quotes "about 40% faster" than LAQ++ at 64K rather than 75% — different draft and look-ahead settings, but the two numbers are not reconciled in the text.

**Cross-family drafts (Appendix E.5, Llama-3.1-70B 4-bit target, LongBench "All"):**

| Method | Draft | All |
|---|---|---|
| SnapKV / LAQ++ | — | 47.75 / 48.43 |
| SpecKV | Llama-3.2-3B (in-family) | 48.80 |
| SpecKV | Qwen2.5-0.5B (cross) | 48.26 |
| SpecKV | Qwen2.5-1.5B (cross) | 48.73 |
| SpecPrefill | Llama-3.2-1B | 48.37 |
| SpecPC | Llama-3.2-1B (in-family) | 48.44 |
| SpecPC | Qwen2.5-0.5B (cross) | **42.04** |
| SpecPC | Qwen2.5-1.5B (cross) | **46.48** |

Cross-tokenizer handling: SpecKV de-tokenizes the draft output and re-tokenizes with the target; SpecPC aggregates token scores to *word* level before re-tokenizing and swaps the chat template. SpecKV is robust because it only consumes the draft's *text*; SpecPC "relies on similar attention patterns between the draft and target models, a condition that may not hold in cross-family setups." The 0.5B cross-family draft loses 6.4 points (single-doc QA 56.84 → 36.73); the 1.5B one loses 2.0.

> [!warning] Contradiction
> [[cross-family-speculative-prefill-upasani-2026]] reports that attention-based importance "transfers reliably" across families (Qwen3-1.7B draft for Llama-3.1-8B within 0-1 points of an in-family Llama-3.2-1B draft). This paper finds cross-family SpecPC loses 2-6 points on the same benchmark family with Qwen2.5 drafts for Llama-3.1-70B. Differences that may explain it: draft generation (Qwen2.5 vs Qwen3), target scale (70B vs 8B), token-to-word aggregation vs text-span reassembly with delimiters, absolute budget (1,024 tokens) vs percentage keep rates, and n_lookahead = 1 vs 8. Neither paper tests the other's configuration; treat "does cross-family drafting work?" as *usually yes, but draft choice and score-transfer details matter*.

**Where it breaks.** (1) Extreme prompt compression (SpecPC-256: 32-34 vs 48-50 dense) — per-head KV dropping tolerates far smaller budgets than prompt-level dropping because prompt compression removes the same tokens for every layer and head. (2) Small cross-family drafts for SpecPC. (3) A "reasonably accurate draft model is still required" (Limitations). (4) Very long outputs or very large n_lookahead may hurt SpecKV. (5) The "consistently outperform" claim is not universal: LAQ++ beats SpecKV on Qwen2.5-72B at C_max = 512/1,024 (49.08/49.59 vs 47.44/47.73) and on Qwen2.5-14B at 256 (45.05 vs 44.09); SnapKV edges SpecKV at C_max = 1,024 on both 32B and 70B (Tables 9-10). The advantage is real and largest at small budgets, and the appendix says so ("in most settings, especially with small C_max").

**Answers to the wiki's standing questions.**
- *Positions:* **SpecPC re-indexes contiguously** (explicitly, to exceed the target's context limit). **SpecKV keeps original positions** — the target prefills the entire prompt with normal RoPE and only the cache is pruned afterward, as in SnapKV. SpecKV-PC inherits SpecPC's contiguous IDs and then prunes.
- *Cross-family draft:* works for **SpecKV** (draft text only), degraded for **SpecPC** (draft attention) — 2-6 points on LongBench with Qwen2.5 drafts for a Llama-3.1-70B target; see the contradiction callout.
- *Dropped-token KV:* **SpecPC** — never computed by the target; gone for the entire generation. **SpecKV** — computed during (sparse) prefill, then **evicted before decode**; also gone for the generation, but the full-prompt prefill did happen (so a serving system could snapshot it). **LAQ++**, the contrast case, keeps the full cache and therefore saves no memory. No multi-turn or follow-up-question evaluation in the paper.
- *Aggressive keep ratios:* on RULER at 64K, C_max = 1,024 (1.6% keep) still leads all prompt-compression baselines (Figure 5b); on LongBench, SpecPC at 1,024 tokens retains ~91% (Qwen-32B) / ~97% (Llama-70B) of dense; at 2,048 tokens 97% / 100%; SpecPC-256 collapses to ~67%.

## Related

- [[speculative-prefill]] — the technique family (no concept page yet); SpecPC is its most refined instance and SpecKV its KV-cache-side sibling.
- [[speculative-prefill-liu-2025]] — the direct ancestor; used here as a baseline (re-implemented) and generalized.
- [[cross-family-speculative-prefill-upasani-2026]] — the contradicting cross-family result.
- [[speculative-decoding]] — same draft model, different job; the paper is explicit that approximate inference reduces compute while speculative decoding does not.
- [[methods-to-speed-up-prompt-processing]] — draft-based importance as a unifying lens on sparse prefill, KV dropping and prompt compression.
- [[kv-cache]] — SpecKV drops to 256 entries per head after prefill; SpecKV-PC also shrinks what is prefilled.
- [[prompt-compression]] — SpecPC vs LLMLingua-2, CPC, R2C; see [[llmlingua-2-pan-2024]] (LLMLingua-2 scores 30.90/33.63 here, far below the draft-attention methods).
- [[attention-mechanism]] / [[attention-sink]] — importance is attention received from late-window and look-ahead queries; StreamingLLM-style sinks are one of the KV baselines.
- [[positional-encoding]] — contiguous re-indexing (SpecPC) vs original positions (SpecKV).
- [[flash-attention]] / [[minference-dynamic-sparse-attention-jiang-2024]] / [[minimax-sparse-attention-lai-2026]] — FlashAttention-2 throughout; MInference's Vertical-Slash kernels for SpecKV's sparse prefill; FlexPrefill cited as the adaptive-sparsity relative.
- [[long-context-llm]] — RULER 4K-64K and LongBench are the benchmarks; MileBench for multimodal.
- [[grouped-query-attention]] — KVCache-Factory extended by repeating KV per head and averaging across KV heads.
- [[vllm]] — used for latency where attention maps are not needed; [[qwen]], [[llama]] — the model families.

## References

- [arXiv:2506.08373](https://arxiv.org/abs/2506.08373) — v3, 2 Feb 2026; ICLR 2026.
- Builds on [[speculative-prefill-liu-2025]] (arXiv:2502.02789) and Lookahead Q-Cache (LAQ++, Wang et al. 2025).
