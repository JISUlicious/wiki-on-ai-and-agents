---
title: Sparse Prefill
type: concept
created: 2026-09-03
updated: 2026-09-03
sources:
  - speculative-prefill-liu-2025.md
  - lazyllm-dynamic-token-pruning-fu-2024.md
  - minference-dynamic-sparse-attention-jiang-2024.md
  - draft-based-approximate-inference-galim-2025.md
  - token-sparse-attention-jo-2026.md
status: complete
importance: high
---

# Sparse Prefill

The umbrella for every technique that makes prefill cheaper by **not doing all of it** — as opposed to doing all of it faster ([[flash-attention]], FP8) or not doing it at all ([[prefix-caching]]). Three families, orthogonal and composable, distinguished by *what* is skipped and *who decides*:

| Family | What is skipped | Who decides | Cuts FFN too? | Concept |
|---|---|---|---|---|
| **Speculative prefill** | whole tokens | a separate small **draft model** | yes | [[speculative-prefill]] |
| **Token pruning** | whole tokens | the **target's own** early layers | yes | [[token-pruning]] |
| **Sparse attention** | entries of the attention matrix; every token stays | per-head pattern, offline or online | **no** | [[sparse-attention]] |

The first two shrink the sequence, so every downstream matmul — attention *and* FFN — runs on fewer rows. The third keeps the sequence and sparsifies only the O(N²) term. That difference is the whole decision rule.

## The decision rule: attention's share of prefill grows with N

Per the [[llm-inference-roofline-yuan-2024|roofline analysis]], prefill FLOPs are dominated by the FFN and projections at ~1,024–1,215 OPs/byte; the QK matmul is a smaller, memory-bound piece. But that is at a few thousand tokens. Attention's share is quadratic and the rest is linear, so the share climbs:

| Context | Attention share of prefill (8B-class) | Source |
|---|---|---|
| ~10K | **~¼** | derived from [[claa-cross-layer-attention-aggregation-mcdanel-2026\|CLAA]]'s −39% TTFT at 10% keep |
| 128K | **~70–80%** | Amdahl-solved from [[token-sparse-attention-jo-2026]]'s ×1.36 attention → ×1.27 TTFT |
| 300K–1M | **>90%** | stated by [[minference-dynamic-sparse-attention-jiang-2024\|MInference]]; ~97% implied at 1M |

So:

- **Below ~64K, sparse attention is Amdahl-bounded and often net-negative.** Token Sparse Attention measures ×0.97 at 8K on A100; MInference at 8K–10K runs at **×0.45–0.47** — *slower than dense* — because pattern-search overhead exceeds the attention it saves. Only token-dropping helps here, because only it shrinks the FFN.
- **Above ~300K, sparse attention is the lever.** MInference reaches **10× end-to-end at 1M** on a single A100 (30 min → 3 min), which no token-dropping method approaches, and it keeps every token.
- **In between, compose.** [[draft-based-approximate-inference-galim-2025|SpecKV-PC]] cascades a draft-picked token set with vertical-slash sparse attention; Token Sparse Attention stacks on MInference/FlexPrefill for ×2.76 → ×3.49 attention speedup.

## What every token-dropping method shares

**Dropped tokens never receive KV entries.** They are absent for the whole generation and for subsequent turns. [[speculative-prefill-liu-2025|Liu et al.]] state it in their limitations: multi-turn *"can potentially fail"* without recomputation. Only the dynamic pruners — [[lazyllm-dynamic-token-pruning-fu-2024|LazyLLM]] and [[sliminfer-dynamic-token-pruning-long-2025|SlimInfer]] — can *revive* a dropped token at a later decode step; [[discovering-the-gems-in-early-layers-shi-2024|GemFilter]], SpecPrefill, and [[probe-and-skip-self-predictive-token-skipping-wu-2026|SPTS]] cannot.

**Quality is task-shaped, not uniform.** At 10% keep on a 70B target, [[speculative-prefill-liu-2025|SpecPrefill]] retains 98.5% of LongBench and code completion *improves* (52.33 → 63.33) — but RULER **aggregation** tasks (common/frequent-word extraction) collapse 99.7 → 77.9 at 4K, summarization loses ~16% relative, and GemFilter's NarrativeQA falls 32.0 → 20.7. Retrieval and QA survive aggressive keep ratios; anything that needs *broad coverage* or *counting* does not. The keep ratio you can afford is a property of the task, not the method.

**Positional handling is unsettled.** Liu et al., oMLX, LazyLLM, SlimInfer and SPTS keep dropped tokens' *original* position IDs — Liu calls this "crucially essential" for retrieval and counting. [[cross-family-speculative-prefill-upasani-2026|Upasani et al.]], Galim's SpecPC, and GemFilter **re-index contiguously**, claiming negligible impact — with **no ablation reported** by any of them. See [[rope-scaling]] for why the difference is not cosmetic.

## Where it sits among prefill levers

Sparse prefill is the **"shrink it"** tier of [[methods-to-speed-up-prompt-processing]], alongside [[prompt-compression]] — but it differs from LLMLingua-style compression in two ways that matter: the selector is *attention*, not a separate learned compressor, so it is training-free and task-agnostic; and (for the positional variants) the kept tokens keep their positions, so the model still knows *where* in the document each survivor came from.

It is orthogonal to [[prefix-caching]] (skip repeated content), [[chunked-prefill]] (schedule prefill against decode — which *spends* prefill, not saves it), [[speculative-decoding]] (accelerates decode, not prefill), and precision.

## Related

- [[speculative-prefill]] · [[token-pruning]] · [[sparse-attention]] — the three families.
- [[methods-to-speed-up-prompt-processing]] — the synthesis this page is a tier of.
- [[kv-cache]] · [[prefix-caching]] · [[prompt-compression]] · [[chunked-prefill]] · [[flash-attention]] · [[llm-inference-roofline-yuan-2024]]
- [[omlx-specprefill-github-2026]] — a production implementation on [[apple-silicon|Apple Silicon]].

## References

- [[speculative-prefill-liu-2025]] · [[cross-family-speculative-prefill-upasani-2026]] · [[draft-based-approximate-inference-galim-2025]] · [[lazyllm-dynamic-token-pruning-fu-2024]] · [[discovering-the-gems-in-early-layers-shi-2024]] · [[sliminfer-dynamic-token-pruning-long-2025]] · [[probe-and-skip-self-predictive-token-skipping-wu-2026]] · [[minference-dynamic-sparse-attention-jiang-2024]] · [[token-sparse-attention-jo-2026]] · [[claa-cross-layer-attention-aggregation-mcdanel-2026]]

## Related 2026 sources

- [[fast-prefill-fpga-sparse-attention-jayanth-2026]] — sparse attention implemented in silicon — a **5.4 TOPS FPGA beating a 222 TOPS GPU** because the kernel is memory-bound, which is this cluster's decision rule stated in hardware
- [[uniprefill-block-wise-dynamic-sparsification-fan-2026]] — token pruning that **cascades through every layer**, and the rule that on hybrid architectures sparse attention is capped at ≤1.11× while token dropping cuts GEMM everywhere. 1.21–2.26× TTFT on LLaMA-3.1-8B, 1.08–1.68× on Qwen3-Next-80B-A3B; **negative at 4K–8K on hybrids** (−5%/−4%), confirming this page's net-loss rule
- [[trianglemix-decoding-time-contribution-sparsity-he-2025]] — a *new kind* of sparsity — attention blocks that score high at prefill but contribute nothing to decoding (gradient probe). Static triangle mask, training-free; **15.3× on the kernel at 128K but 1.14× end-to-end at 32K**, which is this page's attention-share rule measured (34% at 32K, 70% at 128K)
- [[sale-low-bit-estimation-sparse-attention-prefill-ji-2025]] — INT4 QK estimation to choose blocks; ≥3.36× on attention past 64K, 1.39× at 8K (≈1.07× TTFT); needs INT4 Tensor Cores
- [[pop-prefill-only-pruning-he-2026]] — the **model-axis** family this page did not have: keep every token, skip the last third of layers at prefill only. Training-free, 1.36× at 2K prompts
- [[fast-forward-predictive-ffn-sparsity-prefill-gautam-2026]] — same axis, FFN width instead of depth; trained predictor + compensator; 1.44× at 2–4K, <6% LongBench loss
- [[sts-speculative-token-sparsity-yu-2026]] — **does not belong here** — decode-only despite the name; filed under [[sparse-attention]] with [[specattn-speculating-sparse-attention-shah-2025]]. Recorded so the misfiling is not repeated
