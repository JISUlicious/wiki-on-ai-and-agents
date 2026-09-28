---
title: "Cross-Family Speculative Prefill: Training-Free Long-Context Compression with Small Draft Models"
type: source
created: 2026-09-03
updated: 2026-09-03
sources: []
arxiv_id: "2603.02631"
year: 2026
authors:
  - Shubhangi Upasani
  - Ravi Shanker Raju
  - Bo Li
  - Mengmeng Ji
  - John Long
  - Chen Wu
  - Urmish Thakker
  - Guangtao Wang
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Cross-Family Speculative Prefill: Training-Free Long-Context Compression with Small Draft Models

**Shubhangi Upasani, Ravi Shanker Raju, Bo Li, Mengmeng Ji, John Long, Chen Wu, Urmish Thakker, Guangtao Wang** (SambaNova AI) — [arXiv:2603.02631](https://arxiv.org/abs/2603.02631), cs.CL. v1 3 Mar 2026, v3 13 Mar 2026 (this summary follows v3). No venue given. All latency numbers are measured on SambaNova's **RDU** (Reconfigurable Dataflow Unit), not GPUs.

> [!note] Speculative prefill is not speculative decoding
> [[speculative-decoding]] uses a draft to propose *output* tokens that the target verifies — a decode-phase accelerator. Speculative prefill ([[speculative-prefill-liu-2025]]) uses a draft's attention to decide which *input* tokens the target should prefill at all — a prefill/TTFT accelerator. This paper keeps the second mechanism unchanged and asks only whether the draft may come from a **different model family with a different tokenizer**.

## Summary

The original SpecPrefill assumed an in-family draft that shares the target's tokenizer (Llama-3.1-8B for Llama-3.1-70B/405B). Many frontier targets have no small sibling — DeepSeek-V3.1/R1, Kimi-K2 — and agentic stacks mix families for cost and availability reasons. This paper runs the **unmodified** SpecPrefill importance estimator (8-step look-ahead, max over layers and heads, mean over look-ahead tokens, 1D average pooling, top-K chunks) with drafts from Qwen3 (0.6B/1.7B/4B) and Llama-3.x (1B/8B) against targets from Llama-3.1, Qwen3 and DeepSeek (V3, V3.1, R1). Because draft and target tokenizations differ, the selected chunks are **mapped back to text spans, joined with a delimiter token, re-tokenized by the target, and given fresh contiguous position IDs** — the target sees a shorter prompt, not a positionally sparse one.

The headline finding is that attention-based importance **transfers across families**: on LongBench v2 an in-family Llama-3.2-1B draft and a cross-family Qwen3-1.7B draft give the Llama-3.1-8B target 29.4 vs 29.6 at 25% keep and 30.6 vs 31.2 at 50% (baseline 31.2), and for a Qwen3-8B target both in- and cross-family 1-2B drafts land at 97-109% of baseline. On RULER at 128K, compressing to 16K (12.5% keep) with a Llama-3.1-8B draft lets DeepSeek-V3 score **93.36 vs an 80.0 full-prompt baseline** — but that baseline is itself handicapped by SnapStream KV-cache compression, which the RDU needs to fit 128K at all. TTFT for that configuration falls from **46 s to ~2.5 s (~18×)**, or 4.3 s at a 32K budget.

The paper's honest boundary: the "90-100% retention" claim holds for QA-style long-context tasks but **not** for (a) small drafts whose native context is shorter than the input (Qwen3-1.7B at 32K on 248K-token LongBench v2 prompts for DeepSeek-R1: 79-82% of baseline), or (b) repository-scale code debugging at aggressive keep rates (84-88% at 15%).

## Key points

**What changes vs. the original SpecPrefill (Algorithm 1).**

1. Tokenize the prompt text with the **draft** tokenizer (length S). Run the draft with N = 8 look-ahead steps, collecting attention tensors of shape [N, L, S, H].
2. Importance = max over layers L and heads H, then mean over N (identical to Liu et al.). All layers, all heads.
3. 1D average pooling with **kernel 13**, partition into contiguous chunks of **32 draft tokens** (128 for code), average within chunk, keep top-K chunks such that the kept fraction ≈ ρ.
4. **Keep rate ρ is defined in the target's token space**: ρ = L̂_target / L_target, chosen so the compressed prompt is 8K, 16K, 24K or 32K target tokens (multiples of the RDU's 4K-token KV block). ρ is computed per sample; tables report the benchmark average.
5. Merge adjacent selected chunks into spans, **map spans back to text**, concatenate with a task-specific delimiter marking each discontinuity — `[...]` for LongBench/RULER, `// omitted` for code — and re-tokenize with the **target** tokenizer. Target-side length deviates slightly from L̂_target because of tokenizer mismatch.
6. Assign **new contiguous position IDs** p = (0, 1, ..., |u|-1) and run the target's normal prefill and decode.

Step 6 is the deliberate departure from [[speculative-prefill-liu-2025]], which restores original position IDs. The authors say contiguous re-indexing avoids cross-tokenizer position alignment and "has negligible impact on task accuracy across all evaluated benchmarks" — but **no ablation supporting that sentence is reported**; every table uses contiguous IDs. Given that Liu et al. found original IDs "crucially essential" for position-sensitive synthetic tasks, treat this as an unresolved difference between the two papers, plausibly explained by the delimiter tokens standing in for the missing positions.

**LongBench v2 (Table 1), greedy decoding, average input 242-260K tokens:**

| Target | Draft | Keep | easy | hard | short | medium | long | Acc | vs. full |
|---|---|---|---|---|---|---|---|---|---|
| Llama-3.1-8B-Instruct | full prompt | — | 31.2 | 31.2 | 36.1 | 29.3 | 26.9 | 31.2 | — |
| | Llama-3.2-1B-Ins (in-family) | 25% | 30.2 | 28.9 | 39.4 | 23.7 | 24.1 | 29.4 | 94% |
| | Qwen3-1.7B (cross) | 25% | 32.8 | 27.7 | 38.3 | 26.5 | 21.3 | 29.6 | 95% |
| | Llama-3.2-1B-Ins (in-family) | 50% | 32.8 | 29.3 | 40.0 | 26.5 | 23.1 | 30.6 | 98% |
| | Qwen3-1.7B (cross) | 50% | 31.8 | 30.9 | 37.8 | 28.4 | 25.9 | 31.2 | 100% |
| | Qwen3-0.6B (cross) | 10% | 33.3 | 28.0 | 35.6 | 28.8 | 23.1 | 30.0 | 96% |
| Qwen3-8B | full prompt | — | 30.7 | 28.3 | 39.4 | 23.7 | 23.1 | 29.2 | — |
| | Qwen3-1.7B (in-family) | 25% | 35.4 | 29.6 | 37.8 | 27.9 | 29.6 | 31.8 | 109% |
| | Llama-3.2-1B-Ins (cross) | 25% | 32.3 | 26.0 | 34.4 | 24.2 | 26.9 | 28.4 | 97% |
| | Qwen3-1.7B (in-family) | 50% | 33.9 | 26.7 | 38.9 | 22.3 | 27.8 | 29.4 | 101% |
| | Llama-3.2-1B-Ins (cross) | 50% | 35.9 | 26.4 | 39.4 | 25.1 | 24.1 | 30.0 | 103% |
| DeepSeek-R1 | full prompt | — | 66.1 | 53.4 | 62.2 | 54.4 | 59.3 | 58.3 | — |
| | Qwen3-4B-Ins-2507 (262K ctx) | 6% | 57.8 | 50.5 | 64.4 | 45.1 | 50.9 | 53.3 | 91% |
| | Llama-3.1-8B-Ins (128K ctx) | 6% | 55.2 | 53.4 | 60.6 | 45.6 | 60.2 | 54.1 | 93% |
| | Qwen3-1.7B (32K ctx) | 3% | 50.0 | 43.4 | 50.6 | 38.1 | 53.7 | 45.9 | 79% |
| | Qwen3-1.7B (32K ctx) | 6% | 54.2 | 44.1 | 56.1 | 39.5 | 50.9 | 47.9 | 82% |
| | Qwen3-1.7B (32K ctx) | 10% | 53.6 | 42.8 | 58.3 | 37.7 | 46.3 | 46.9 | 80% |

("vs. full" is computed here from the table; the paper reports raw accuracies.) For DeepSeek-R1, 3/6/10% keep corresponds to ~8K/16K/24K compressed tokens. The 8B-scale targets are near-indistinguishable between in-family and cross-family drafts at matched size; the deciding variable for the 250K-token R1 prompts is the **draft's native context length**, not its family — a 32K-context draft cannot attend over the whole prompt, so its saliency estimate degrades, and accuracy sits ~20 points below the long-context drafts at the same keep rate.

**Code Debug, InfiniteBench (Table 2), Llama-3.1-8B-Instruct draft, ~110K-token inputs:**

| Target | Keep | Compressed length | Acc | vs. full |
|---|---|---|---|---|
| DeepSeek-V3.1 | full | — | 67.51 | — |
| | 20% | ~22K | 64.72 | 95.9% |
| | 15% | ~16K | 59.13 | 87.6% |
| DeepSeek-R1-0528 | full | — | 74.37 | — |
| | 30% | ~33K | 70.30 | 94.5% |
| | 25% | ~28K | 68.02 | 91.5% |
| | 15% | ~16K | 62.44 | 84.0% |

Appendix A.5 attributes the drop to code's distributed syntactic/semantic dependencies: cutting low-saliency spans breaks the local context needed to spot the injected bug. The authors conclude the method is "structure-dependent" and suggest dependency-aware or function-coverage constraints.

**LongBench v1 (Table 3), compressed to a fixed 4K target tokens (avg keep 45%, avg input ~8.8K):** Llama-3.1-8B-Instruct with a Qwen3-1.7B draft 54.29 vs 55.31 full (98%); Qwen3-8B with a Llama-3.2-1B draft 55.98 vs 57.34 (98%). Per-task the only notable losses are LCC/RepoBench code completion (−3 to −4 points) and pass.ret.zh.

**RULER 128K → 16K (Table 4), DeepSeek-V3 target, 12.5% keep:**

| Config | niah s1 | s2 | s3 | mk1 | mk2 | mk3 | mv | mq | qa_1 | qa_2 | Avg |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Full prompt (with SnapStream KV compression) | 100 | 100 | 62.8 | 89.2 | 99.6 | 32.2 | 98.75 | 99.25 | 61.2 | 55.0 | 80.0 |
| SP w/ Qwen3-4B-Ins-2507 | 100 | 100 | 100 | 100 | 83.4 | 76.4 | 99.6 | 100 | 70.46 | 66.84 | 89.67 |
| SP w/ Llama-3.1-8B-Instruct | 100 | 100 | 100 | 99.8 | 99.6 | 96.8 | 99.2 | 100 | 70.74 | 67.43 | **93.36** |

Read this table carefully: the "full prompt" row is *not* a clean dense baseline. DeepSeek-V3 on the RDU is capped at a 32K context, so 128K inference required SnapStream KV-cache compression, which throws away long-range information (hence 32.2 on multikey-3). Speculative prefill sidesteps that by shrinking the prompt *before* the target sees it. The comparison shows that a long-context draft plus a 32K-context target beats a 128K target with lossy cache compression — a real deployment result, but not evidence that SpecPrefill improves on unconstrained dense inference.

**Latency (Section 5.2, RDU).** 128K-token RULER prompts: full-prompt TTFT **46 s**; compressed to 16K, **~2.5 s (~18×)**; compressed to 32K, **4.3 s**. The paper says it uses "the same TTFT measurement methodology commonly used for GPU-based systems" but does not break out draft-model time vs. target prefill, nor report GPU numbers.

**Context-window extension (Section 5.4).** Because the draft does the long-context reading, a target deployable only at 32K can consume 128K inputs; more generally a 256K-context draft can front a 128K target. This reframes speculative prefill as a *context-extension* primitive as much as a latency one.

**Robustness to draft choice (Section 5.3).** 1-1.7B drafts suffice for 8B targets; 4B-8B drafts matter mainly for very long inputs. The authors' explanation: tokens that matter for reasoning (numbers, entities, key instructions) attract attention consistently across architectures trained on similar data, so importance "depends mainly on task priors and semantic structure".

**Where it breaks.** (1) Draft native context shorter than the input (Qwen3-1.7B on 250K prompts: ~80% retention). (2) Code debugging below ~20% keep (84-88%). (3) Anything Liu et al. already flagged — aggregation/counting tasks, information-dense short prompts — is not re-evaluated here. (4) The RDU-only latency numbers and the SnapStream-handicapped RULER baseline limit how far the efficiency claims transfer to GPU serving.

**Answers to the wiki's standing questions.**
- *Positions:* **re-indexed contiguously** (p = 0..|u|-1) after text-level reassembly with delimiter tokens; original positions are *not* preserved. This is the opposite choice from Liu et al., justified by tokenizer mismatch and asserted (without a reported ablation) to cost nothing.
- *Cross-family draft:* **yes, it works.** Qwen3 drafts for Llama targets, Llama drafts for Qwen3 and DeepSeek targets, all within a few points of in-family drafts at matched draft size. What matters more than family is the draft's native context length relative to the input.
- *Dropped-token KV:* **does not exist in the target.** The target receives only the compressed text and builds KV for it alone. Dropped spans are gone for the whole generation; the paper does not address multi-turn or follow-up questions about dropped content. (Since compression happens at the text level, a serving system *could* retain the full prompt and re-prefill on demand, but nothing here does so.)
- *Aggressive keep ratios:* 10% keep with a Qwen3-0.6B draft keeps 96% on LongBench v2 for an 8B target; 12.5% on RULER beats the (handicapped) baseline; 6% on 250K prompts keeps 91-93% with long-context drafts; 15% on code drops to 84-88%. A clean 20% point is only given for Code Debug (95.9%).

**Abstract claims vs. body.** The abstract's "largely retains 90-100%" is true of the QA-style tables but the body itself reports 79-82% (Qwen3-1.7B draft for DeepSeek-R1) and 84-88% (code at 15%); "largely" is load-bearing. The "up to ~18× TTFT" is supported only by one RDU configuration against a SnapStream-compressed baseline. The abstract lists DeepSeek among the "draft-target combinations"; DeepSeek models appear **only as targets**, never as drafts.

## Related

- [[speculative-prefill]] — the technique (no concept page yet); this paper generalizes its draft-selection assumption.
- [[speculative-prefill-liu-2025]] — the original SpecPrefill; identical scoring, different positional handling and draft-family assumption.
- [[speculative-decoding]] — decode-phase draft/verify; not what this is.
- [[methods-to-speed-up-prompt-processing]] — where cross-family speculative prefill slots into the wiki's prefill-speedup synthesis.
- [[prompt-compression]] — this variant is literally text-level compression (spans + delimiters + re-tokenize), closer in interface to [[llmlingua-jiang-2023]] than Liu et al.'s positional [[sparse-prefill|sparse prefill]].
- [[kv-cache]] / [[paged-attention]] — 4K-token KV blocks on the RDU dictate the 8K/16K/24K/32K compressed lengths; SnapStream cache compression is the baseline's handicap.
- [[positional-encoding]] — contiguous re-indexing vs. original-ID restoration is the open question between the two papers.
- [[long-context-llm]] — LongBench v1/v2, RULER, InfiniteBench Code Debug; the context-extension reading (32K target consuming 128K inputs).
- [[attention-mechanism]] — the transfer claim is that attention-derived saliency is a property of the task and data, not of the architecture.
- [[agentic-context-engineering]] — the motivating workload: heterogeneous model stacks, repeated long-prompt calls.
- [[qwen]] / [[llama]] / [[deepseek]] — the three families paired here; [[sambanova]] — the authors' affiliation and the RDU hardware.
- [[draft-based-approximate-inference-galim-2025]] — a contemporaneous generalization of draft-based importance beyond prefill.

## References

- [arXiv:2603.02631](https://arxiv.org/abs/2603.02631) — v3, 13 Mar 2026.
- Builds directly on [[speculative-prefill-liu-2025]] (arXiv:2502.02789).
