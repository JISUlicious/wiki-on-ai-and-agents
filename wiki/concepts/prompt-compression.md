---
title: Prompt Compression
type: concept
created: 2026-07-27
updated: 2026-07-27
sources:
  - llmlingua-jiang-2023.md
  - llmlingua-2-pan-2024.md
status: complete
importance: medium
---

# Prompt Compression

**Prompt compression** shortens the prompt itself before it reaches the model, so fewer tokens enter prefill at all. Unlike [[prefix-caching]] — which *reuses* prefill work — compression *removes* it, trading accuracy for tokens.

## LLMLingua — perplexity-guided, coarse-to-fine

[[llmlingua-jiang-2023|Jiang et al. (EMNLP 2023)]]: a budget controller drops whole demonstrations first (protecting the instruction at τ=0.85 and question at τ=0.9), then iterative token-level compression in 100-token segments, each conditioned on the already-compressed prefix, using a small LM aligned to the target model's distribution.

GSM8K is the flattering case: **78.85 EM at 2,366 tokens → 79.08 at 446 (5×), 77.41 at 14×, 77.33 at 117 tokens (20×)** — the "20× for ~1.5 points" headline. At 20× the Selective-Context baseline scores 44.20, a **33-point gap**.

> [!warning] The task dependence is severe
> BBH degrades **−8.5 points at 5×** and **−13.2 at 7×**, and everything collapses past ~25–30×. Compression ratios do not transfer across task types — validate on your own workload before trusting a ratio.

**The speedup is net**, which is the thing to check about any compressor: end-to-end V100-32G **8.6 s → 4.9 s (2×) → 2.3 s (5×) → 1.3 s (10×)**, of which LLMLingua's own pass costs only 0.8 / 0.3 / 0.2 s. Cost on GSM8K: **$5.2 → $0.5**.

## LLMLingua-2 — task-agnostic token classification

[[llmlingua-2-pan-2024|Pan et al. (Findings of ACL 2024)]] abandons perplexity for **binary token classification (preserve/discard)** over a bidirectional encoder (XLM-RoBERTa-large), trained on labels **distilled from GPT-4**. GPT-4 is used *offline only*, so inference is a single encoder forward pass.

- MeetingBank QA EM: **87.75 at 3,003 tokens → 86.92 at 970 (3.1×)**, vs LLMLingua's 67.52 at 2.5×
- **3–6× faster than prior compressors, 8× less GPU memory**
- Compressor overhead: **0.4 s**, vs 1.5–2.9 s (LLMLingua) and **15.5 s (Selective-Context)**
- Extractive, so **faithful by construction**; generalizes out of domain despite training only on meeting transcripts

Two honest limits the authors state: it **loses to task-aware LongLLMLingua on LongBench**, and with Mistral-7B it actually *beats* the uncompressed prompt — higher information density helping a model that handles long context poorly.

> [!important] Always check the compressor's own cost
> Selective-Context's compression pass costs **15.5 s** — more than the entire uncompressed inference. A compressor that isn't dramatically cheaper than the model is a **net slowdown**, however good its compression ratio looks.

## Choosing between compression and caching

Being **task-agnostic**, an LLMLingua-2-compressed document is query-independent — so it stays **cacheable** and needn't be recompressed per RAG query. That makes the two techniques composable: compress the static corpus once, then prefix-cache the compressed form.

Related evidence that prompts are highly compressible: [[can-lms-actually-retrieve-in-context-gollapudi-2026|BlockSearch]] finds the **top 0.1% of context tokens carry 50–80% of relevance mass**, and [[recontext-recursive-evidence-replay-zhao-2026|ReContext]] exploits model-internal attention as the selection signal.

## Related

- [[prefix-caching]] · [[long-context-llm]] · [[retrieval-augmented-generation]] · [[quantization-performance]]

## References

- [[llmlingua-jiang-2023]] · [[llmlingua-2-pan-2024]]
