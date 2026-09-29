---
title: "LazyLLM: Dynamic Token Pruning for Efficient Long Context LLM Inference"
type: source
created: 2026-09-03
updated: 2026-09-03
sources: []
arxiv_id: "2407.14057"
year: 2024
authors:
  - Qichen Fu
  - Minsik Cho
  - Thomas Merth
  - Sachin Mehta
  - Mohammad Rastegari
  - Mahyar Najibi
tags:
  - 2024
  - paper
status: complete
importance: high
---

# LazyLLM: Dynamic Token Pruning for Efficient Long Context LLM Inference

**Qichen Fu, Minsik Cho, Thomas Merth, Sachin Mehta, Mohammad Rastegari, Mahyar Najibi** (Apple; Rastegari listed at Meta AI, "work done while working at Apple") — [arXiv:2407.14057](https://arxiv.org/abs/2407.14057), cs.CL, 19 Jul 2024. No venue given.

## Summary

The prefill stage computes the [[kv-cache]] for *every* prompt token before the first output token appears, so time-to-first-token (TTFT) grows with prompt length while decode steps stay cheap. On Llama 2 7B over LongBench (average prompt 3,376 tokens, average generation 68 tokens) the authors measure TTFT at **21× the wall time of one decode step** and **23% of total generation time**. Profiling the attention of prompt tokens with respect to the first generated token (Fig. 2) shows it is very sparse, which motivates the question the paper is built around: are all prompt tokens necessary to generate the first token? The two existing families cannot answer it. KV-cache eviction methods (H2O, SnapKV, Keyformer, DMC) profile importance from the *full* attention maps of the first few generated tokens, so they still pay the full prefill and only accelerate decode. [[prompt-compression]] methods (Selective Context, [[llmlingua-jiang-2023]]) run an LLM to shrink the prompt first, and that overhead makes TTFT *worse*, not better.

**LazyLLM** uses the target model's own attention as the selector — no draft model, no training, no parameter changes. At layer *l* the importance of prompt token *i* is the attention probability that the **last token (position N) pays to token i, averaged over heads**: s_i^l = (1/H) Σ_h A^l_{h,i,N}. Token *i* is pruned at layer *l+1* if s_i^l falls below the k^l-th percentile of the input tokens; once pruned it is excluded from all subsequent layers of that step, so each layer's token set is a subset of the previous layer's. Pruning is **progressive along depth** — more tokens kept in early layers, more pruned toward the end — because a single-layer ablation (Sec. 5.4) shows later layers are consistently less sensitive to pruning than earlier ones. The distinguishing design is that pruning is **dynamic per generation step**: every step, prefill and each decode step alike, re-selects its own subset of the prompt, and a token pruned at step *t* can be **revived** at a later step *t' > t*. The authors call revival "crucial to retain accuracy." Revival is made cheap by an **Aux Cache** that stores the hidden states of pruned tokens whose KV is absent from the next layer's cache; when layer *l+1* needs a token whose KV was never computed, it pulls that token's hidden state from the Aux Cache of layer *l* instead of re-running the token through layers 1..*l*. This guarantees every token is computed **at most once per layer** across the whole generation, so the worst-case runtime is never slower than the baseline. Pruned tokens' KV entries are therefore not discarded — they are simply *never computed* at the layers that pruned them, and are lazily computed on demand if the token is later selected.

Evaluated training-free on **Llama 2 7B and XGen 7B** across all 16 LongBench datasets (6 task categories) on A100s with HuggingFace transformers. On multi-document QA, LazyLLM gives **2.34× TTFT speedup on Llama 2 with score 22.43 → 22.31** and 2.65× on XGen (20.71 → 20.68); [[few-shot-learning|few-shot learning]] 2.19× / 3.42×; synthetic 2.89× / 4.77×; code completion 1.94× / 3.47×. Loosening the pruning schedule reaches **3.0× on multi-document QA at ≤10% accuracy loss**. Because many prompt tokens are never selected at any step, total compute also drops: only 63.94% of Llama 2's multi-doc-QA prompt tokens are ever computed (40.54% for XGen on synthetic tasks), yielding end-to-end generation speedups of 1.01-3.16×. The same tables show the honest edge cases: summarization computes 99.59% of tokens on Llama 2 (1.02× overall speedup), and code completion on Llama 2 loses 1.88 points.

## Key points

**Where this sits among prefill token-pruning methods** — the four distinctions the wiki's synthesis depends on:

| Question | LazyLLM |
|---|---|
| Who selects tokens? | The **target model itself**: each layer's attention row for the last token (mean over heads). Selection cost is part of the same forward pass, not a separate small-model pass as in speculative prefill. |
| Which layer selects? | **Every pruning layer** uses the immediately preceding layer's attention map; pruning is progressive across depth, not a single early-layer decision. Exact pruning-layer positions and percentiles are not disclosed. |
| Static or dynamic? | **Dynamic per step, with revival** — a different subset at prefill and at every decode step; pruned tokens can be re-selected later. |
| Dropped tokens' KV? | **Never computed at the pruning layer**, not evicted. Hidden states of pruned tokens are held in the Aux Cache so a revived token resumes from the layer that dropped it. |
| Prefill FLOPs reduced? | **Yes** — pruned tokens skip attention and FFN in all later layers during prefill, which is what produces the TTFT speedups; it also cuts total generation compute (Table 2). |

**Table 1 — TTFT speedup vs. accuracy on LongBench** (score = macro-average over the datasets in each task; speedup = measured wall time vs. standard inference):

| Task | Method | Llama 2 7B score | TTFT speedup | XGen 7B score | TTFT speedup |
|---|---|---|---|---|---|
| Single-doc QA | Baseline | 25.79 | 1.00× | 25.19 | 1.00× |
| | Static token pruning | 21.89 | 1.18× | 19.30 | 1.61× |
| | Prompt compression | 22.88 | 0.12× | 15.31 | 0.20× |
| | **LazyLLM** | 25.59 | **1.36×** | 25.00 | **1.96×** |
| Multi-doc QA | Baseline | 22.43 | 1.00× | 20.71 | 1.00× |
| | Static token pruning | 19.93 | 2.16× | 17.23 | 2.11× |
| | Prompt compression | 8.42 | 0.13× | 11.56 | 0.19× |
| | **LazyLLM** | 22.31 | **2.34×** | 20.68 | **2.65×** |
| Summarization | Baseline | 24.65 | 1.00× | 24.85 | 1.00× |
| | Static token pruning | 24.59 | 1.33× | 24.46 | 1.65× |
| | **LazyLLM** | 24.75 | **1.46×** | 24.74 | **1.91×** |
| Few-shot learning | Baseline | 62.90 | 1.00× | 56.40 | 1.00× |
| | Static token pruning | 56.54 | 2.16× | 51.93 | 3.17× |
| | Prompt compression | 24.18 | 0.10× | 23.72 | 0.15× |
| | **LazyLLM** | 62.81 | **2.19×** | 56.12 | **3.42×** |
| Synthetic | Baseline | 4.97 | 1.00× | 5.40 | 1.00× |
| | Static token pruning | 2.81 | 2.15× | 3.00 | 4.14× |
| | **LazyLLM** | 4.98 | **2.89×** | 5.66 | **4.77×** |
| Code completion | Baseline | 55.18 | 1.00× | 36.49 | 1.00× |
| | Static token pruning | 37.51 | 1.84× | 32.27 | 2.97× |
| | Prompt compression | 17.45 | 0.49× | 11.38 | 0.69× |
| | **LazyLLM** | 53.30 | **1.94×** | 36.47 | **3.47×** |

A "random token drop" baseline (Random-LTD style, averaged over 5 runs) is also reported and is uniformly worse than static pruning; omitted here for space. The **"static token pruning" baseline is the authors' own construction** — prune once, using the attention scores of the first few layers during prefill — which is essentially the GemFilter recipe ([[discovering-the-gems-in-early-layers-shi-2024]]). Static pruning reaches similar TTFT speedups but loses far more accuracy (e.g. code completion 37.51 vs. 53.30 on Llama 2), which is the paper's evidence that per-step revival matters. The **"prompt compression" baseline is Selective Context (Li et al., 2023)**, not LLMLingua; its TTFT speedup of 0.10-0.49× (i.e. 2-10× *slower*) is the paper's evidence that LLM-driven compression cannot help TTFT.

**Table 2 — cumulative prompt tokens computed and end-to-end speedup** (same settings as Table 1):

| Task | % prompt tokens computed (Llama 2) | (XGen) | Overall generation speedup (Llama 2) | (XGen) |
|---|---|---|---|---|
| Single-doc QA | 87.31 | 89.16 | 1.34× | 1.33× |
| Multi-doc QA | 63.94 | 69.60 | 1.56× | 1.70× |
| Summarization | 99.59 | 96.11 | 1.02× | 1.09× |
| Few-shot learning | 69.98 | 65.30 | 1.28× | 1.59× |
| Synthetic | 63.73 | 40.54 | 1.79× | 3.16× |
| Code completion | 68.57 | 72.61 | 1.01× | 1.16× |

"% computed" is the fraction of prompt tokens that were selected at *any* step by the end of generation — equivalently the final KV-cache size relative to the full prompt (Sec. 5.5). Anything below 100% is compute the baseline spent that LazyLLM never did. FFN cost scales linearly and attention quadratically with this fraction.

**Speed/accuracy frontier (Fig. 5, Llama 2 7B)** — three knobs: number of pruning layers, their positions, and the per-layer pruning percentile. At matched TTFT speedup LazyLLM retains accuracy better than every baseline; 2.34× on multi-doc QA costs ≤1%, 3.0× costs ≤10%.

**Single-layer ablation (Sec. 5.4, Fig. 6)** — pruning once at various depths and ratios, both models: accuracy falls monotonically as fewer tokens are kept at a fixed layer, and pruning at a later layer is always better than the same ratio at an earlier layer. This is the empirical basis for the progressive schedule, and it is also the reason LazyLLM does *not* prune aggressively in the first few layers the way GemFilter does.

**Where it breaks / what is missing**
- The **pruning schedule is not reported** — neither the number and positions of pruning layers nor the percentiles behind Table 1. Results cannot be reproduced from the paper text alone.
- **Summarization gains almost nothing end-to-end** (99.59% of tokens eventually computed, 1.02× overall), because the model keeps reviving tokens across the output; TTFT still improves 1.46× since the first step is sparse.
- **Code completion** is the largest quality loss (Llama 2 −1.88 points, −3.4% relative) even at a modest 1.94× speedup.
- Only **7B models** and only LongBench, whose average prompt is ~3.4K tokens — no evidence at 32K-128K contexts where prefill actually dominates.
- Only relative speedups are given; no absolute latencies, no statement of whether the baseline uses [[flash-attention]], and the memory cost of the Aux Cache (a second hidden-state cache alongside the KV cache) is never quantified.
- Attention-score selection requires materializing the last row of the attention map at each pruning layer, which is incompatible with fused attention kernels that never expose probabilities; the paper does not discuss this.

**Abstract vs. body** — every headline claim is supported: 2.34× multi-doc QA at negligible loss (Table 1), acceleration of both prefill and decode (Tables 1 and 2), and no fine-tuning (identical checkpoints). The abstract's "significantly accelerate the generation" should be read against Table 2, where end-to-end gains are 1.01-1.79× on Llama 2 — the big multipliers are TTFT-only.

## Related

- [[kv-cache]] — LazyLLM changes *when* KV is computed (lazily, on selection) rather than evicting it; the Aux Cache is a companion structure holding hidden states of not-yet-computed tokens.
- [[attention-mechanism]] — the selector is the model's own attention from the last position, so selection cost is a partial forward pass of the target model. Contrast [[speculative-prefill]] (no concept page yet; see [[speculative-prefill-liu-2025]]), which uses a separate small draft model for the same job.
- [[token-pruning]] — the technique family; LazyLLM is the dynamic-with-revival member, [[discovering-the-gems-in-early-layers-shi-2024]] the static-once member.
- [[longbench]] — the benchmark for all results; 16 datasets, 6 tasks, macro-averaged per task.
- [[prompt-compression]] and [[llmlingua-jiang-2023]] — the contrasting approach; Selective Context is shown to make TTFT 2-10× slower because compression itself runs an LLM.
- [[long-context-llm]] — the motivating regime, though the evaluation stays at ~3.4K-token prompts.
- [[llm-inference-roofline-yuan-2024]] — prefill is compute-bound, which is why cutting the token count through the stack translates directly into TTFT.
- [[chunked-prefill]] / [[sarathi-chunked-prefill-agrawal-2023]] — orthogonal: reorganizes prefill for throughput rather than shrinking it.
- [[flash-attention]] — orthogonal exact-attention kernel; note the tension that score-based pruning needs attention probabilities that fused kernels do not expose.
- [[speculative-decoding]] — the decode-side acceleration family the paper explicitly sets aside; the same Apple group's Speculative Streaming is cited.
- [[llama]] — Llama 2 7B is the primary test model.
- [[methods-to-speed-up-prompt-processing]] — where this belongs in the wiki's taxonomy of prefill-acceleration techniques.

## References

- [arXiv:2407.14057](https://arxiv.org/abs/2407.14057)
- LongBench: Bai et al., 2023, arXiv:2308.14508
- Selective Context (the "prompt compression" baseline): Li et al., 2023, arXiv:2310.06201
- Random-LTD (the "random token drop" baseline): Yao et al., 2022, arXiv:2211.11586
