---
title: Cross-Model KV Cache Transfer
type: concept
created: 2026-08-11
updated: 2026-08-11
sources:
  - cross-model-kv-cache-transfer-heo-2026.md
  - droidspeak-kv-cache-sharing-liu-2024.md
builds-on:
  - [[kv-cache]]
status: complete
importance: medium
---

# Cross-Model KV Cache Transfer

Reusing one model's [[kv-cache|KV cache]] in a **different model**, so the receiver skips prefill entirely.

This is a distinct lever from [[prefix-caching]]. Prefix caching reuses KV **within one model** when the *text* repeats. Cross-model transfer reuses KV **across models** when the text is unchanged but the *model* switched — the cost–quality cascade, mid-conversation upgrade, and router-swap cases, where today the receiving model repays the whole prefill from scratch.

## Two approaches, opposite tradeoffs

| | **[[droidspeak-kv-cache-sharing-liu-2024\|DroidSpeak]]** (2024) | **[[cross-model-kv-cache-transfer-heo-2026\|Heo et al.]]** (NVIDIA, 2026) |
|---|---|---|
| Mechanism | reuse most layers verbatim, **recompute ~11% "critical" layers** | learn a **closed-form linear map** from source KV to target KV |
| Model requirement | **identical architecture**, different weights | **matched-KV**: same KV head count and per-head dim — *layer and parameter count may differ* |
| Per-pair setup | offline profiling only | ridge fit per head; **1.01–3.36 B params / 4–12 GB storage per directed pair** |
| Prefill cost | partial (recomputed layers) + **E-cache transfer** (up to 2–4× KV size) | none |
| Speedup | **1.7–3.1×** prefill, 2.1× avg; up to 4× throughput | **4–25×** small→large, **3–7×** large→small |
| Reliability | negligible quality loss across 8 pairs | **4 of 6 pairs** retain 73–98%; **2 degrade sharply** |

The architecture constraint is what separates them: DroidSpeak needs the *same* architecture, which disqualifies it from the 14B→32B cascading case that motivates the NVIDIA work. The NVIDIA paper cites it on exactly this point.

## Why a linear map works at all

Cross-model KV turns out to carry substantial **linear structure** between matched-KV pairs. On Qwen3 14B→32B, a single source layer explains **56% of variance in target keys and 32% in values**, rising to **79% / 65%** when several source layers are concatenated.

The method is three steps, per head:
1. **Select the k most predictive source layers** and concatenate their KV as input.
2. **Strip RoPE from keys before mapping**, so the fit is *position-free* and therefore reusable across context lengths — the step that makes it practical.
3. **Ridge regression** (λ=0.01) on a small calibration set: 500 FineWeb-Edu sequences × 1,024 tokens, stride-4 subsampled to ~128K tokens per target head.

Speedups are real and grow with context (Qwen3 14B↔32B, small→large): **14.0 vs 61.7 ms at 64 tokens (4×)**, **67.8 vs 1154.8 ms at 8K (17×)**, **277.6 vs 6975.3 ms at 32K (25×)**.

> [!note] The abstract's low end is from a different experiment
> The abstract quotes **2.7×**, which comes from the Appendix G sweep; the main text reports 4–25× (S→L) and 3–7× (L→S). Cite the range you mean.

## The finding worth carrying beyond this topic

**Error *placement* determines retention, not error magnitude.**

Calibration fit quality is **anti-correlated** with downstream accuracy retention (**r = −0.20**), while attention-output cosine similarity predicts it (**r = +0.57**). The clearest case: Llama 3.1 8B→70B fits at **R² = 0.84** yet retains **94%** small-to-large and only **37%** large-to-small — same mapper quality, opposite outcomes by direction.

The nonlinear MLP variant wins **not by shrinking error but by redistributing it into attention-irrelevant directions**.

That is a general caution about proxy metrics: a regression that fits well can still destroy the thing you care about, because what matters is *which subspace* the residual lands in. It rhymes with [[quantization-quality]]'s finding that perplexity understates capability damage, and with [[verifier]]'s point that a proxy is only as good as its alignment with the target.

## Where it breaks

- **Both failures are Ministral 3 → 14B**: 8B→14B (41.6% Avg) and 3B→14B (44.2%).
- **GSM8K is fragile everywhere** — 18.2% even on a Tier-1 pair. Arithmetic reasoning is the first thing to go.
- **Storage is per *directed* pair** (4–12 GB), so a cascade over N models needs mappers for each ordered pair you intend to use.
- Multi-turn drift is measured for the **ridge** mapper only: on CoQA, small→large widens 1.7 pp over ten turns, large→small drifts 0.33 pp/turn.

> [!warning] The MLP variant has no reported latency
> Its evaluation is quality-only (HellaSwag retention on 4 pairs). Table 5's timings are ridge-only. So "a nonlinear MLP can be significantly faster than re-prefill" is **not** a measured claim in this paper.

## Related

- [[kv-cache]] · [[prefix-caching]] — the within-model counterpart.
- [[grouped-query-attention]] — GQA shrinks KV but *not* embeddings, which is why DroidSpeak's E-cache transfer costs 2–4× the KV.
- [[methods-to-speed-up-prompt-processing]] — where this sits among prefill levers.
- [[learned-orchestration]] · [[when-is-routing-meaningful-huot-2026]] — the routing/cascade setting that motivates it.

## References

- [[cross-model-kv-cache-transfer-heo-2026]] · [[droidspeak-kv-cache-sharing-liu-2024]]
