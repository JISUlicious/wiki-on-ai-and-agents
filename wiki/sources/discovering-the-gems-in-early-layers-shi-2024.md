---
title: "Discovering the Gems in Early Layers: Accelerating Long-Context LLMs with 1000x Input Token Reduction (GemFilter)"
type: source
created: 2026-09-03
updated: 2026-09-03
sources: []
arxiv_id: "2409.17422"
year: 2024
authors:
  - Zhenmei Shi
  - Yifei Ming
  - Xuan-Phi Nguyen
  - Yingyu Liang
  - Shafiq Joty
tags:
  - 2024
  - paper
status: complete
importance: high
---

# Discovering the Gems in Early Layers: Accelerating Long-Context LLMs with 1000x Input Token Reduction (GemFilter)

**Zhenmei Shi, Yifei Ming, Xuan-Phi Nguyen, Yingyu Liang, Shafiq Joty** (Salesforce AI Research; UW-Madison; HKU) — [arXiv:2409.17422](https://arxiv.org/abs/2409.17422), cs.CL, 25 Sep 2024 (v1 only). No venue given. Code: `SalesforceAIResearch/GemFilter`.

## Summary

KV-cache eviction (H2O, SnapKV) can shrink a 128K-token [[kv-cache]] to ~1,024 entries with negligible loss, but it only helps the *iterative generation* phase: the full prompt still has to pass through every layer to produce the attention maps those methods score with, so prompt computation — the dominant cost as inputs lengthen — is untouched. GemFilter asks whether prompt computation itself can be cut. Its empirical claim is that an LLM "finds" the query-relevant tokens in its **early-to-middle layers, before it starts generating**: for LLaMA 3.1 8B Instruct the last row of the attention matrix in any of layers 13-19 already points at the needle in a Needle-in-a-Haystack prompt (Fig. 2, Fig. 5). The method exploits this by running the model twice.

**Algorithm 1** takes a filter layer *r* and a budget *k*. Pass 1 runs only layers 1..*r* over the full *n*-token prompt, takes the layer-*r* query of the **last token** and the layer-*r* keys of all tokens, and scores every token by the **pre-softmax inner product Q_n·K^T summed across heads** (a max-pool of kernel size 5 is applied, as in SnapKV). The top-*k* indices form a single index set *J*, which is re-sorted into original order so that `<bos>` stays first. Pass 2 then runs the **entire model, from scratch, on the k-token subsequence T_J** with a standard generation function — RoPE is recomputed over positions 1..*k*, so the maximum positional distance the model sees drops from *n+t* to *k+t*. Everything from pass 1 is thrown away; nothing about the *n−k* dropped tokens survives, and they can never be revived. Selection is therefore **static and query-conditioned**, performed by the **target model's own layer r** — a partial forward pass of the big model, not a separate draft model. Defaults are *r*=13 of 32 for LLaMA 3.1 8B, *r*=19 of 40 for Mistral Nemo 12B, *r*=19 of 32 for Phi 3.5 Mini 3.8B, and *k*∈{1024, 2048, 4096}. Theorem 3.3 gives prompt-phase time Θ(rhn²d) versus Θ(mhn²d) for standard attention *and* for SnapKV/H2O, and prompt-phase memory rw + 2hnd versus mw + 2mhnd — only *r* layers of weights and a single layer's K/V need be resident.

Results are strongest exactly where a single query token is the whole task. On **Needle in a Haystack** (120K tokens for LLaMA 3.1, 60K for Mistral Nemo, two A100-40GB) GemFilter-1024 scores **0.887 vs. 0.841 for full attention and 0.749 for SnapKV-1024** on LLaMA 3.1, and **0.838 vs. 0.486 / 0.494** on Mistral Nemo — beating the *uncompressed* model, which the authors attribute to the shortened positional range in pass 2. On the 14 English **LongBench** datasets it is "comparable": averages within 0.5-2.2 points of full KV at *k*=1024 and above full KV on Mistral Nemo at *k*=2048 (47.11 vs. 46.36), while compressing the input to **8% (k=1024) or 32% (k=4096)** of its tokens. Wall-clock, with FlashAttention on an H100-80GB, the prompt phase is **2.4× faster than both SnapKV and standard attention**, with **30% less GPU memory than SnapKV and 70% less than standard attention** (Fig. 3, Fig. 6 — figure-only, no absolute times in the text). The **"1000×" in the title is one demonstration**, not a benchmark setting: a 108,172-token NIAH prompt reduced to *k*=100 (Fig. 1); every reported evaluation uses *k*≥1024.

## Key points

**Where this sits among prefill token-pruning methods** — the four distinctions the wiki's synthesis depends on:

| Question | GemFilter |
|---|---|
| Who selects tokens? | The **target model's own layer r** — pre-softmax Q_n·K^T from the last prompt token, summed over heads, one shared index set for all layers and heads (SnapKV/H2O keep m·h sets). Selection costs an r-layer forward pass of the big model over the full prompt. |
| Which layer selects? | A **single fixed early-to-middle layer**: 13/32 (LLaMA 3.1 8B), 19/40 (Mistral Nemo 12B), 19/32 (Phi 3.5 Mini). Chosen as the first layer whose top-k reliably covers the needle; performance is flat from layer 13 to 25 on LLaMA. |
| Static or dynamic? | **Static, decided once** before generation; the same k tokens serve every decode step. No revival is possible. |
| Dropped tokens' KV? | **Discarded entirely** — pass 1's layer 1..r KV is not reused, and dropped tokens are never seen by pass 2. |
| Prefill FLOPs reduced? | **Yes**: full-prompt work shrinks from m to r layers (theoretical ceiling m/r ≈ 2.46× for LLaMA 3.1 8B; measured 2.4×), plus a full but short pass on k tokens. This is genuine prefill acceleration, unlike SnapKV/H2O which the paper shows share the Θ(mhn²d) prompt cost of standard attention. |

**Complexity (Theorem 3.3)** — *n* prompt length, *m* layers, *h* heads, *d* head dim, *k* selected tokens, *t* generated tokens, *w* per-layer weight memory:

| | Standard attention | SnapKV / H2O | GemFilter |
|---|---|---|---|
| Prompt-phase time | Θ(mhn²d) | Θ(mhn²d) | **Θ(rhn²d)** |
| Generation-phase time | Θ(mh(nt+t²)d) | Θ(mh(kt+t²)d) | Θ(mh(k²+t²)d) |
| Prompt-phase memory | mw + 2mhnd | mw + 2hnd + 2mhkd | **rw + 2hnd** |
| Generation-phase memory | mw + 2mh(n+t)d | mw + 2mh(k+t)d | mw + 2mh(k+t)d |

For n=128K, k=t=1024 the prompt-phase time ratio is m : m : r. Note the k² term in GemFilter's generation phase: that is pass 2's own prefill over the k selected tokens.

**Needle in a Haystack (k=1024, average score over lengths × depths)**:

| Model (max length) | Full KV | SnapKV-1024 | GemFilter-1024 |
|---|---|---|---|
| LLaMA 3.1 8B Instruct (120K) | 0.841 | 0.749 | **0.887** |
| Mistral Nemo 12B Instruct (60K) | 0.486 | 0.494 | **0.838** |
| Phi 3.5 Mini 3.8B Instruct (App. C.3) | 0.851 | 0.864 | **0.910** |

H2O is excluded from NIAH and timing because its cumulative-attention scoring cannot run under FlashAttention.

**LongBench averages (14 English datasets; Table 1)** — code-completion datasets (LCC, RepoBench-P) are *not* evaluated:

| Model | Full KV | H2O-4096 | SnapKV-1024 | GemFilter-1024 | SnapKV-2048 | GemFilter-2048 | SnapKV-4096 | GemFilter-4096 |
|---|---|---|---|---|---|---|---|---|
| LLaMA 3.1 8B | 36.72 | 33.63 | 35.25 | 34.50 | 35.80 | 35.87 | 36.44 | 36.09 |
| Mistral Nemo 12B | 46.36 | 43.69 | 44.71 | 45.14 | 45.42 | **47.11** | 46.04 | 46.79 |
| Phi 3.5 Mini 3.8B | 34.62 | 30.31 | 33.68 | 32.74 | 34.20 | 31.69 | 34.53 | 33.35 |

Averages hide large per-dataset swings. On LLaMA 3.1 at k=1024, NarrativeQA falls **32.02 → 20.71** (SnapKV-1024 holds 31.98), GovReport 34.52 → 30.37, and PassageRetrieval 97.73 → 92.22, while HotpotQA/2WikiMQA/Musique *improve* (16.23 → 19.12, 16.05 → 17.01, 11.22 → 13.01). On Phi 3.5 NarrativeQA drops 27.51 → 16.57 and PassageCount 7.17 → 0.2 at k=1024.

**Filter-layer ablation (LLaMA 3.1 8B, k=1024, LongBench average; Table 2)**: layer-1 23.50, layer-7 27.47, layer-12 28.03, **layer-13 34.50**, layer-14 34.30, layer-19 34.12, layer-25 33.94, layer-31 30.04. Accuracy rises sharply between layers 12 and 13, plateaus through 25, then falls — the authors read this as three stages (early layers search, early-to-middle layers identify, late layers prepare output). The selected layer is the first one whose top-k covers the needle (Fig. 5), and they report it is consistent across inputs.

**Setup** — HuggingFace v4.43, greedy decoding, no training or randomness anywhere. NIAH/LongBench on A100-40GB; timing on H100-80GB with FlashAttention. SnapKV/H2O use a 32-token observation window (their recommended setting); GemFilter has no window because it scores from a single last-token query.

**Where it breaks / what is missing**
- **Non-monotonic in k.** On Phi 3.5 GemFilter-2048 (31.69) is *worse* than GemFilter-1024 (32.74), and its PassageRetrieval score falls as k grows (81.12 → 69.5 → 65.56). More selected tokens does not reliably mean better output, which the paper does not explain.
- **Static selection is blind to what later decode steps need.** Summarization and narrative QA — tasks that need broad coverage rather than one needle — lose the most; there is no revival path because dropped tokens are gone. Contrast [[lazyllm-dynamic-token-pruning-fu-2024]], whose per-step re-selection exists precisely for this failure mode.
- **Last-token-as-query assumption.** Scoring uses only the final prompt token's query at layer r. Prompts where the instruction precedes a long document, multi-question prompts, and multi-turn histories are not evaluated.
- **Speedup is capped at m/r** by construction (~2.46× for LLaMA 3.1 8B) regardless of k or context length; it cannot scale the way per-layer progressive pruning can.
- **Timing evidence is figures only.** No absolute latencies, and the text does not state the context length behind the 2.4× / 30% / 70% figures.
- Evaluated only at 3.8B-12B with 128K-context instruct models; no code-completion results; LongBench prompts are short enough that k=4096 already keeps 32% of the input, so the LongBench numbers are not a long-context speed test.

**Abstract/title vs. body** — "2.4× speedup and 30% memory reduction vs. SOTA" and "significantly outperforms standard attention and SnapKV on NIAH" are supported (Fig. 3/6, Fig. 4). "Comparable performance on LongBench" is true of averages but not of narrative QA or summarization at k=1024. The title's **"1000× input token reduction" is supported only by the single Fig. 1 example** (108,172 → 100 tokens); the largest benchmarked reduction is ~120× (120K → 1024 on NIAH) and 8-32% retention on LongBench.

## Related

- [[kv-cache]] — the paper's framing is that KV *eviction* (SnapKV, H2O) leaves prompt computation untouched; GemFilter cuts the prompt pass itself, then runs pass 2 with a k-token cache.
- [[attention-mechanism]] — selection uses raw last-token Q·K^T at one layer, so the model's own attention is the retriever; contrast [[speculative-prefill]] (no concept page yet; see [[speculative-prefill-liu-2025]]), which delegates that to a small draft model.
- [[token-pruning]] (no page yet) — GemFilter is the canonical *static, decide-once* member of this family; [[lazyllm-dynamic-token-pruning-fu-2024]] is the dynamic-with-revival member and reports a "static token pruning" baseline built on this same recipe.
- [[longbench]] (no page yet) — 14 English datasets here (no code completion); also Needle in a Haystack.
- [[flash-attention]] — all timing runs use it; the paper notes H2O's cumulative scoring is incompatible with fused attention, which is why H2O is absent from NIAH and timing.
- [[positional-encoding]] / [[rope-scaling]] — pass 2 recomputes RoPE over k+t positions; the authors credit this shortened range for GemFilter beating the uncompressed model on NIAH.
- [[long-context-llm]] — the target regime; NIAH runs at 60K-120K tokens.
- [[prompt-compression]] — Selective Context (LDLG) must keep ~50% of tokens; GemFilter claims comparable quality at ~1%.
- [[attention-sink]] — `<bos>` is preserved by re-sorting J into original order.
- [[llm-inference-roofline-yuan-2024]] — why cutting the prompt pass from m to r layers maps directly onto wall-clock TTFT.
- [[llama]] — LLaMA 3.1 8B Instruct is the primary model.
- [[methods-to-speed-up-prompt-processing]] — where this belongs in the wiki's taxonomy of prefill-acceleration techniques.

## References

- [arXiv:2409.17422](https://arxiv.org/abs/2409.17422)
- Code: https://github.com/SalesforceAIResearch/GemFilter
- SnapKV: Li et al., 2024, arXiv:2404.14469 · H2O: Zhang et al., 2023 (NeurIPS 36)
- Needle in a Haystack: Kamradt, 2024, `gkamradt/LLMTest_NeedleInAHaystack`
- LongBench: Bai et al., 2023, arXiv:2308.14508
