---
title: "LLMLingua-2: Data Distillation for Efficient and Faithful Task-Agnostic Prompt Compression"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - llmlingua-2-pan-2024.md
arxiv_id: "2403.12968"
year: 2024
venue: Findings of ACL 2024
authors:
  - Zhuoshi Pan
  - Qianhui Wu
  - Huiqiang Jiang
  - Menglin Xia
  - Lili Qiu
tags:
  - 2024
  - paper
status: complete
importance: high
---

# LLMLingua-2: Data Distillation for Efficient and Faithful Task-Agnostic Prompt Compression

**Zhuoshi Pan, Qianhui Wu, Huiqiang Jiang, et al.** (Tsinghua University and Microsoft Corporation; Pan interned at Microsoft) — [arXiv:2403.12968](https://arxiv.org/abs/2403.12968), Findings of ACL 2024.

## Summary

LLMLingua-2 keeps the goal of [[llmlingua-jiang-2023|LLMLingua]] — fewer tokens entering prefill — but replaces the mechanism. LLMLingua-1 ranked tokens by **perplexity from a causal small LM** (LLaMA-2-7B / Alpaca-7B). The authors argue that metric is doubly wrong: information entropy is only a proxy, not the compression objective, and a causal LM sees **unidirectional context only**. LLMLingua-2 instead reframes compression as **binary token classification (preserve / discard)** over a bidirectional **Transformer encoder** — XLM-RoBERTa-large for the main model, multilingual BERT for `LLMLingua-2-small`. At inference the model emits a preserve-probability per word; the top `τ·N` words are kept **in their original order**. Because it is purely extractive, the output is guaranteed faithful to the input (no hallucinated or rephrased content, unlike generative compression), and it is **task-agnostic** — no question or task conditioning, so a document compressed once is reusable across queries, which matters a lot in RAG-style pipelines where task-aware compression would recompress the same document per query.

The training signal comes from **data distillation from GPT-4**. The authors prompt GPT-4-32k to compress MeetingBank meeting transcripts "as short as possible while retaining as much information as possible," deliberately *not* specifying a target ratio (information density varies by genre and speaker, and GPT-4 does not obey ratio instructions well). Long texts are split into **chunks of ≤512 tokens ending on a period** before compression, because GPT-4 compresses very long contexts too aggressively and loses information. An annotation algorithm then aligns GPT-4's compressed output back onto the original tokens to produce preserve/discard labels, coping with GPT-4's three misbehaviors (word variation in tense/voice/plurality, reordering, and newly-introduced words) via bidirectional fuzzy matching. Two quality-control metrics — **Variation Rate** (fraction of compressed words absent from the original, a hallucination proxy) and an alignment-quality metric — filter bad examples. The resulting extractive compression dataset is released.

The distillation cost is a **one-time offline expense**: GPT-4 is used to build the dataset, never at inference. At inference the compressor is a single forward pass through an encoder of BERT-base or RoBERTa-large size — which is why the compression overhead nearly disappears and the end-to-end speedups are unambiguously net.

## Key points

- **In-domain (MeetingBank, GPT-3.5-Turbo target).** QA exact match: original prompt 87.75 at 3,003 tokens; **LLMLingua-2 86.92 at 970 tokens (3.1×)** and LLMLingua-2-small 85.82 at 984 tokens (3.0×) — versus LLMLingua 67.52 and Selective-Context 66.28 at only 2.5×. Summary BERTScore 88.27 vs 88.96 for the original. A ~20-point jump over the perplexity-based predecessor at a higher compression ratio.
- **Latency, and this is the headline (Table 5, MeetingBank, V100-32G).** End-to-end **14.9 s uncompressed → 9.4 s at 2× (1.6×), 7.5 s at 3× (2.1×), 5.2 s at 5× (2.9×)**. The compressor itself costs only **0.5 / 0.4 / 0.4 s**, versus **2.9 / 2.1 / 1.5 s for LLMLingua** and **15.9 / 15.6 / 15.5 s for Selective-Context**. That last figure is the important cautionary number: Selective-Context's compression pass alone costs *more than the entire uncompressed inference*, so its "compression" is a net end-to-end slowdown. LLMLingua-2 is **3×–6× faster than existing compression methods** and also cuts GPU memory for the compression step by **8×**.
- **Out-of-domain generalization holds** despite training only on meeting transcripts. On GSM8K (5× compression) LLMLingua-2 scores **79.08 EM at 457 tokens vs 78.85 full-shot at 2,366 tokens** — matching LLMLingua (79.08) and far above Selective-Context (53.98); at 14× it reaches 77.79 (LLMLingua 77.41). On BBH: 70.02 at 3× and 61.94 at 5×, again matching or beating LLMLingua and crushing Selective-Context (54.27 / 54.02). LongBench and ZeroSCROLLS show the same pattern.
- **The BERT-base-size model is nearly as good.** LLMLingua-2-small matches or slightly beats the LLaMA-2-7B-based baselines across all out-of-domain benchmarks — a ~60× smaller compressor doing better work because it is trained on the compression objective directly.
- **Honest about its limit**: LLMLingua-2 **loses to task-aware LongLLMLingua on LongBench**, because task-aware methods exploit the question. The authors argue task-agnostic compression is the right trade for deployment generality and for reuse of a compressed document across queries.
- **Sometimes beats the original prompt.** With **Mistral-7B-v0.1** as target, LLMLingua-2 outperforms the uncompressed prompt (e.g. MeetingBank QA 76.22 vs 66.95 original, at 3.0× compression). The authors attribute this to Mistral-7B handling long context poorly — higher information density helps more than raw context does.
- **Faithfulness is structural, not measured after the fact.** Extractive token classification cannot invent text; GPT-4 can also reconstruct the original prompt from an LLMLingua-2-compressed version, which the authors take as evidence of no essential information loss.
- **Composable with LLMLingua-1.** The token-classification compressor can be dropped into LLMLingua's coarse-to-fine framework in place of the perplexity-based ITPC module, keeping the budget controller, reaching **~15× compression** on multi-demonstration/multi-document prompts.
- **Ablations** confirm both the chunk-wise (≤512-token) distillation and the ratio-free instruction design are load-bearing for the final quality.

## Concepts & entities

- [[prompt-compression]] — the direct successor formulation: task-agnostic, extractive, token-classification-based, distilled from GPT-4 rather than read off perplexity.
- [[transformer-architecture]] — the compressor is an *encoder* (XLM-RoBERTa-large / mBERT); bidirectional context is the paper's core architectural argument against causal-LM perplexity scoring.
- [[long-context-llm]] — evaluated on LongBench and ZeroSCROLLS; also demonstrates that a shorter, denser prompt can beat a long one on models weak at long context.
- [[in-context-learning]] — GSM8K/BBH few-shot prompts are the out-of-domain generalization test.
- [[chain-of-thought]] — CoT prompts are named as a primary driver of prompt bloat and are preserved at 5×–14× compression.
- [[retrieval-augmented-generation]] — the motivating case for task-agnostic compression: a document compressed once serves every query, whereas task-aware methods recompress per query.
- [[prefix-caching]] — orthogonal complement; compression shrinks the prefill workload while caching avoids repeating it, and a task-agnostic (query-independent) compressed document stays cacheable.

## References

- [arXiv:2403.12968](https://arxiv.org/abs/2403.12968)
- Code and dataset: aka.ms/LLMLingua-2
