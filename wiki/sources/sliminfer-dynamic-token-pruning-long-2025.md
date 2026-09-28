---
title: "SlimInfer: Accelerating Long-Context LLM Inference via Dynamic Token Pruning"
type: source
created: 2026-09-03
updated: 2026-09-03
sources: []
arxiv_id: "2508.06447"
year: 2025
authors:
  - Lingkun Long
  - Rubing Yang
  - Yushi Huang
  - Desheng Hui
  - Ao Zhou
  - Jianlei Yang
tags:
  - 2025
  - paper
status: complete
importance: high
---

# SlimInfer: Accelerating Long-Context LLM Inference via Dynamic Token Pruning

**Lingkun Long, Rubing Yang, Yushi Huang, Desheng Hui, Ao Zhou, Jianlei Yang** (Beihang University; HKUST) — [arXiv:2508.06447](https://arxiv.org/abs/2508.06447), cs.CL, v1 8 Aug 2025, v2 24 Nov 2025. No venue given. Code: `Longxmas/SlimInfer`. Implementation is explicitly built on top of [[lazyllm-dynamic-token-pruning-fu-2024]]'s codebase.

## Summary

The paper's diagnosis of prior work splits it into three camps that each miss something. Sparse-attention prefill methods (MInference, FlexPrefill) skip attention blocks but still push the **full sequence of hidden states through every layer**, so the FFN — a large share of prefill FLOPs — is untouched. [[kv-cache]] eviction methods (StreamingLLM, H2O, SnapKV, PyramidKV) target decode, and their eviction is **irreversible**, which hurts tasks with long-range dependencies. LazyLLM prunes dynamically but **keeps the whole KV cache resident on the GPU**, so it saves compute without saving memory. SlimInfer's motivating observation is an **information diffusion** effect: on LLaMA 3.1 8B, deleting the hidden state of the critical prompt token "278" at layer 25 still yields the right answer, while deleting it at layer 5 does not (Fig. 2). The attention band from the answer token to that prompt token forms around layer 13; by the time it is truncated at layer 25 the token's content has already spread into its neighbours. Hence two design rules — keep everything in early layers, prune aggressively (even originally important tokens) in later ones.

Mechanically, the prompt is cut into **fixed 64-token blocks**; generated tokens are never pruned. Early **Preserve layers** keep all blocks; at each **Slim layer** (layers 10/20/30 for LLaMA 3.1 8B retaining 8K/4K/2K tokens; layers 9/18/26 for Qwen2.5 7B retaining 12K/6K/4K) the active block set is re-selected **immediately after that layer's attention**, using that layer's own Q and K: each block is split into 8-token **token units** whose keys are mean-pooled into a representative key; the query is the mean of the **last w=4 tokens' queries** (end of prompt in prefill, most recent decoded tokens in decode); block score = max over units of the head-averaged dot product. The first block is always kept as an [[attention-sink]]; the top-k blocks form the candidate set that flows unchanged until the next Slim layer. Because only active blocks have hidden states past a Slim layer, **both attention and FFN skip pruned blocks** — this is what makes it a genuine prefill-FLOP reduction rather than KV compression. Selection is **dynamic per inference step**: every decode step re-scores blocks against the recent decoding context, and **pruned blocks' KV is offloaded to CPU rather than discarded**, so a block can be revived when it becomes relevant again. Because the decision at layer *l* is fixed before layer *l*'s FFN and layer *l+1*'s QKV projection run, the KV swap (offload + prefetch on a separate CUDA stream) can be overlapped with that compute **without any predictor** — the paper's contrast with InfiniGen (SVD rehearsal) and AttentionPredictor (a trained CNN). An overlap-aware rule only triggers a swap when the candidate set overlaps the previous active set by less than γ=0.9; otherwise the old set is reused.

Accuracy is measured on all 17 LongBench datasets (including code) on an H200; speed on a **single RTX 4090 (24 GB)**, batch 1, 16 decoded tokens, against a FlashAttention-2 baseline. LLaMA 3.1 8B: **48.08 → 47.65 average** (LazyLLM at the same pruning layers with 50% retention: 47.23; MInference 47.17; FlexPrefill 46.38); Qwen2.5 7B: 47.76 → 47.38 (LazyLLM 45.48). At 32K context TTFT speedup is **up to 2.53×** and end-to-end **1.88×** (1.60× without the asynchronous KV manager); Qwen2.5 reaches 2.14× / 1.84×. Prompt-KV GPU memory falls **20.3% at 8K to 56.6% at 32K** (4.00 → 1.73 GB). The honest edge of the data: the E2E speedup is exactly **1.00× at 8K** and on a Jetson AGX Orin the 4K case is *slower* than the baseline (0.98× TTFT, 0.93× E2E) — with 8K tokens retained at the first Slim layer, short prompts are never pruned but still pay the block-scoring overhead.

## Key points

**Where this sits among prefill token-pruning methods** — the four distinctions the wiki's synthesis depends on:

| Question | SlimInfer |
|---|---|
| Who selects tokens? | The **target model itself**: at each Slim layer, that layer's own keys (token-unit mean-pooled) dotted with the mean query of the last 4 tokens. No draft model; selection cost is a cheap block-scoring pass inside the same forward. |
| Which layer selects? | **Three Slim layers** (10/20/30 of 32 for LLaMA 3.1 8B; 9/18/26 of 28 for Qwen2.5 7B), each using its *own* Q/K after its attention, with nested budgets 8K→4K→2K (LLaMA) or 12K→6K→4K (Qwen). Early layers never prune. |
| Static or dynamic? | **Dynamic per step, with revival** — the active set is re-scored at every prefill/decode step against the recent context; an overlap threshold γ=0.9 suppresses swaps when the set barely changes. |
| Dropped tokens' KV? | **Kept, offloaded to CPU**, and prefetched back when a block is re-selected. Note the paper never states how a block pruned *during prefill* obtains KV for the layers it skipped; since the code is built on LazyLLM, the Aux-Cache-style lazy recompute is the likely path, but the text only says "offloaded ... restored". |
| Prefill FLOPs reduced? | **Yes** — pruned blocks' hidden states leave the sequence, so attention *and* FFN at every later layer run on 8K/4K/2K tokens instead of N. Back-of-envelope for a 32K prompt on LLaMA (linear terms only): (10·32K + 10·8K + 10·4K + 2·2K)/(32·32K) ≈ 0.43 of baseline FFN/projection work, before the quadratic attention savings — consistent with the measured 2.53× TTFT. |

**LongBench averages (17 datasets; Table 1)**:

| Model | Full KV (FA2) | LazyLLM | MInference | FlexPrefill | **SlimInfer** |
|---|---|---|---|---|---|
| LLaMA 3.1 8B Instruct | 48.08 | 47.23 | 47.17 | 46.38 | **47.65** |
| Qwen2.5 7B Instruct | 47.76 | 45.48 | 47.25 | 44.61 | **47.38** |

LazyLLM is run at SlimInfer's pruning layers with 50% retention per layer; FlexPrefill at γ=0.95; MInference with its official per-head patterns. Per-dataset, the largest SlimInfer losses are Qwen MuSiQue 30.16 → 27.25 and LLaMA PassageCount 7.43 → 6.30 / MultiFieldQA 55.05 → 53.82; several datasets improve slightly (LLaMA MuSiQue 30.78 → 30.95, LCC 63.12 → 63.65, SAMSum 43.92 → 44.36). FlexPrefill's synthetic-task collapse (PassageRetrieval 99.5 → 82.0 on LLaMA, 100 → 75 on Qwen) is the clearest failure among baselines.

**Accuracy vs. E2E latency at 32K, LLaMA 3.1 8B, 16 output tokens (Table E)** — the Pareto data behind Fig. 6:

| Method (setting) | LongBench avg | E2E (s) | E2E speedup |
|---|---|---|---|
| Full KV (FlashAttention-2) | 48.08 | 5.561 | 1.00× |
| MInference | 47.17 | 5.935 | 0.94× |
| FlexPrefill γ=0.99 / 0.98 / 0.95 / 0.90 | 47.19 / 46.87 / 46.38 / 45.64 | 4.949 / 4.690 / 4.403 / 4.231 | 1.12-1.31× |
| LazyLLM 50% / 45% / 40% / 35% retention | 47.23 / 46.90 / 46.66 / 46.12 | 4.364 / 4.124 / 3.893 / 3.655 | 1.27-1.52× |
| SlimInfer (16K,8K,4K) | **47.98** | 3.457 | 1.61× |
| SlimInfer (12K,6K,4K) | 47.90 | 3.119 | 1.78× |
| SlimInfer (10K,5K,3K) | 47.70 | 3.028 | 1.84× |
| SlimInfer (8K,4K,2K) — main setting | 47.65 | 2.959 | **1.88×** |

Read against LazyLLM specifically: at *higher* accuracy (47.65 vs. 46.12) SlimInfer is 0.7 s faster than LazyLLM's most aggressive 35% setting. The paper attributes LazyLLM's gap to per-token (rather than block) bookkeeping and to keeping the full KV resident. MInference is *slower* than dense FlashAttention-2 end-to-end at this length on a 4090.

**End-to-end speedup vs. context length, LLaMA 3.1 8B, RTX 4090 (Table 4)**:

| | 8K | 16K | 24K | 28K | 32K |
|---|---|---|---|---|---|
| SlimInfer w/o async KV | 1.00× | 1.18× | 1.37× | 1.48× | 1.60× |
| SlimInfer w/ async KV | 1.02× | 1.29× | 1.53× | 1.79× | **1.88×** |

TTFT speedup (Fig. 5, figure-only) rises with length to 2.53× at 32K; the text says SlimInfer "far outperforms the baselines at ≥24K and remains on par with them in other cases."

**Prompt KV-cache GPU memory, LLaMA 3.1 8B (Table 2, GB)**:

| | 8K | 16K | 24K | 28K | 32K |
|---|---|---|---|---|---|
| Full KV | 1.00 | 2.00 | 3.00 | 3.50 | 4.00 |
| SlimInfer | 0.80 | 1.11 | 1.42 | 1.58 | 1.73 |
| Saving | 20.3% | 44.5% | 52.6% | 54.9% | 56.6% |

FlexPrefill and MInference save nothing here (full cache at every layer); LazyLLM saves compute but not memory.

**Edge device — Jetson AGX Orin 32 GB, LLaMA 3.1 8B, (8K,4K,2K), seconds (Table F)**; MInference/FlexPrefill could not run (no Triton on Jetson):

| Context | TTFT full → Slim | TTFT speedup | E2E full → Slim | E2E speedup |
|---|---|---|---|---|
| 4K | 12.49 → 12.76 | **0.98×** | 18.22 → 19.56 | **0.93×** |
| 8K | 26.16 → 22.78 | 1.15× | 34.51 → 33.34 | 1.04× |
| 16K | 57.05 → 35.31 | 1.62× | 72.71 → 59.38 | 1.22× |
| 24K | 93.31 → 51.01 | 1.83× | 115.60 → 85.62 | 1.35× |
| 28K | 113.35 → 59.29 | 1.91× | 138.62 → 86.58 | 1.60× |
| 32K | 134.73 → 69.57 | 1.94× | 163.10 → 96.06 | 1.69× |

**[[needle-in-a-haystack|Needle in a Haystack]] (LLaMA 3.1 8B, 50% pruning at layers 10/20/30)**: 0.986 vs. 0.984 for full KV — essentially unchanged.

**Ablations (LLaMA 3.1 8B)**
- *Pruning depth* (single prune, fixed total retention; Fig. 7): accuracy rises as the prune is delayed to middle layers, then MultiFieldQA and Qasper fall sharply if it is delayed further (too few tokens left for late-layer reasoning), while PassageRetrieval stays flat. Same shape as LazyLLM's Sec. 5.4 and GemFilter's layer sweep — early pruning breaks diffusion, late pruning starves the answer.
- *Block scoring* (Table 3, avg over MuSiQue/PassR/HPQA plus full-benchmark avg): avg-pooled keys 47.45, max-pooled 47.44, token-unit max-of-means **47.65**.
- *Swap threshold γ* (Table G, 32K): 0.99 → 47.67 / 3.057 s; 0.95 → 47.67 / 3.040 s; **0.90 → 47.65 / 2.959 s**.
- *Block size*: latency falls monotonically with larger blocks; accuracy peaks at 64 and degrades both smaller (32) and larger.
- *Query window w*: peak at 4; w=1 drops notably; w≥8 degrades gradually.

**Where it breaks / what is missing**
- **No gain below ~8K-16K tokens** — 1.00-1.02× E2E at 8K, and a net *slowdown* at 4K on Jetson. The first Slim layer keeps 8K tokens, so shorter prompts pay scoring and bookkeeping overhead for nothing. LongBench's average prompt is well under 8K, which makes the Table 1 accuracy numbers a test of a mostly-unpruned model.
- **Revival path for prefill-pruned blocks is unspecified.** "Offloaded to CPU ... restored" presumes the KV exists; for blocks pruned at layer 10 during prefill, KV at layers 11-32 was never computed. [[probe-and-skip-self-predictive-token-skipping-wu-2026]] describes both LazyLLM and SlimInfer as "recompute the pruned tokens during decoding, leading to substantial generation slowdown" — consistent with lazy recompute, and with SlimInfer's TTFT gains (2.53×) being much larger than its E2E gains (1.88×).
- **Block granularity** — a 64-token block is kept or dropped whole; token-unit scoring refines the *score*, not the *decision*.
- **Hardware split** — accuracy on an H200, latency on a 4090 and a Jetson; no server-GPU latency, no batching, no comparison of the CPU-offload path under PCIe contention.
- Only 7-8B instruct models; the pruning schedule (layers and budgets) is hand-set per model with no rule for choosing it.

**Abstract vs. body** — "up to 2.53× TTFT and 1.88× E2E on a single RTX 4090" is supported (Fig. 5 / Table 4) but the 2.53× exists only in a figure. "Without sacrificing performance on LongBench" holds on averages (−0.43 and −0.38); Qwen MuSiQue loses 2.9 points. "Reducing both memory usage and I/O costs" is supported for memory (Table 2); I/O cost is only shown indirectly through the async-vs-sync gap (1.60× → 1.88×).

## Related

- [[lazyllm-dynamic-token-pruning-fu-2024]] — direct ancestor and codebase; SlimInfer adds block granularity, current-layer Q·K scoring with a recent-token query window, CPU offload of pruned KV, and predictor-free prefetching. Beats it on both axes in Table E.
- [[discovering-the-gems-in-early-layers-shi-2024]] — the static alternative; not a baseline here, but the "information diffusion" analysis is the same layer-13 phenomenon GemFilter exploits, read the opposite way (SlimInfer keeps early layers dense *because* diffusion happens there).
- [[probe-and-skip-self-predictive-token-skipping-wu-2026]] — successor that re-evaluates SlimInfer as a baseline and argues per-step recompute of revived tokens is what caps its decode speed.
- [[token-pruning]] (no page yet) — the family; SlimInfer is the dynamic-with-revival-plus-offload member.
- [[kv-cache]] — pruned blocks' KV lives on the CPU rather than being evicted; the async manager is a KV-offload scheduler.
- [[attention-sink]] — the first prompt block is unconditionally kept for this reason.
- [[attention-mechanism]] — selection is a dot product of pooled keys against a pooled recent query at the pruning layer; contrast [[speculative-prefill]] (no concept page yet; see [[speculative-prefill-liu-2025]]), which uses a separate draft model.
- [[flash-attention]] — the dense baseline (FlashAttention-2); all methods run on top of it.
- [[longbench]] (no page yet) — all 17 datasets used, plus Needle in a Haystack.
- [[long-context-llm]] — the regime where the method pays off (≥16K-24K tokens).
- [[llm-inference-roofline-yuan-2024]] — why removing hidden states (FFN + attention) beats sparsifying attention alone on prefill.
- [[llama]], [[qwen]] — LLaMA 3.1 8B Instruct and Qwen2.5 7B Instruct.
- [[minference-dynamic-sparse-attention-jiang-2024]] — sparse-attention baseline: slower than dense FlashAttention-2 end-to-end at 32K on a 4090 (Table E) and saves no KV memory, because it still pushes every hidden state through every layer.
- [[methods-to-speed-up-prompt-processing]] — where this belongs in the wiki's taxonomy of prefill-acceleration techniques.

## References

- [arXiv:2508.06447](https://arxiv.org/abs/2508.06447)
- Code: https://github.com/Longxmas/SlimInfer
- LazyLLM: Fu et al., 2024, arXiv:2407.14057 · MInference: Jiang et al., 2024, arXiv:2407.02490 · FlexPrefill: Lai et al., ICLR 2025
- InfiniGen (Lee et al., 2024) and AttentionPredictor (Yang et al., 2025b) — the predictor-based prefetchers SlimInfer positions itself against
