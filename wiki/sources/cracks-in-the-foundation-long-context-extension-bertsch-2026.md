---
title: "Cracks in the Foundation: Seemingly Minor Architectural Choices Impact Long Context Extension"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.10296"
venue: COLM 2026
authors:
  - Amanda Bertsch
  - Luca Soldaini
  - Matthew R. Gormley
  - [[graham-neubig]]
  - Hannaneh Hajishirzi
  - Kyle Lo
  - Dirk Groeneveld
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Cracks in the Foundation: Seemingly Minor Architectural Choices Impact Long Context Extension

**Amanda Bertsch, Luca Soldaini, Matthew R. Gormley, [[graham-neubig|Graham Neubig]], Hannaneh Hajishirzi, Kyle Lo, Dirk Groeneveld** (AI2, [[cmu|CMU]], [[university-of-washington|UW]]) — [arXiv:2608.10296](https://arxiv.org/abs/2608.10296), 10 Aug 2026. **Accepted to COLM 2026**, 29 pages.

## The problem

Nearly every production dense transformer is extended to long context *after* pretraining, by continued training with a modified RoPE base. The paper's question is whether decisions made during **pretraining** — decisions that look free or strictly-better on short-context metrics — silently determine how well that later extension works. The answer is yes, and the damage is invisible to every metric a pretraining team normally watches.

## The method

Four axes are varied in a controlled sweep, holding data, tokenizer, LR schedule, and extension recipe fixed. Each setting **ships in at least one released model family**:

| Axis | Settings compared | Who ships it |
|---|---|---|
| **Normalization** | no QK-norm / layerwise QK-norm / headwise QK-norm (plus prenorm vs. post-sublayer norm) | no QK-norm = Llama 3; layerwise = Olmo 2/3; headwise = Qwen 3, Gemma 3, Marin 32B |
| **[[grouped-query-attention\|GQA]]** | 32 KV heads (full MHA) / 16 / 8 | 8 KV heads = Llama 3; intermediate size adjusted to hold total params constant |
| **Sliding window attention** | none (full attention) / 3 local layers (4K window) per 1 global layer | Olmo 3, Qwen 3, Gemma 3 |
| **Pretraining context length** | 4096 / 8192 | 4K = prior generation; 8K = Llama 3 |

**Setup.** 7–8B dense transformers, **140B pretraining tokens** (Chinchilla-optimal for the size), then context extension by **continued pretraining on 10B tokens** of the Longmino mix with RoPE-θ adjustment and LR annealed to zero, targeting **64K**. Evaluated on **HELMET** (32K and 8K), **RULER** (32K), and LongPPL.

**OlmPool.** The release is the main artifact: **26 comparable 7B models**, **38 checkpoints each**, covering the full pretraining *and* extension trajectories, so the pre-/post-extension pair is available for every configuration. Cost: **~170,000 H100 hours**, which the authors convert to **42.63 metric tons CO₂**.

## Results

**Spread across the pool** — HELMET 32K spans **29.9–56.4**; RULER 32K spans **44.7–67.7**. Same data, same tokenizer, same extension recipe; the only difference is four "minor" architectural knobs.

**The headline.** The abstract says combining three or more of these choices "can drop the performance downstream by **up to 47%**." The Figure 1 caption states the same gap in absolute terms: "the combination of these features can dramatically degrade performance, **up to 26.5 points on HELMET**." These are the same number — 26.5 / 56.4 = 47.0% *relative* to the best configuration. So the 47% is a **relative** drop between the best and worst OlmPool models, not a 47-point swing.

**Per-axis effects (HELMET 32K), and the compounding:**

| Choice | Effect |
|---|---|
| QK-norm (either variant) | −3.8 to −6 points |
| GQA, 8 KV heads vs. 32 | −3.4 points |
| SWA alone (with full attention elsewhere) | −1.1 points |
| **SWA + GQA together** | **−9 points** |
| 4K vs. 8K pretraining context | −1 to −2 points |
| 3+ features combined | severe; worst-case 26.5 points / 47% |

The interaction is the point: SWA costs ~1 point on its own and ~9 in combination with GQA. Consistent with that, the **single most predictive regressor for downstream long-context score is simply the *count* of downweighting features** (R² = 0.67 in-sample, **R² = 0.61 leave-one-out**), beating any per-axis regression.

**Short-context metrics see none of it.** This is the most operationally important table in the paper:

| Predictor | R² vs. HELMET 32K |
|---|---|
| Pretraining loss | 0.29 |
| Loss at end of context extension | **0.06** |
| Held-out perplexity, WikiText (best domain) | 0.39 |
| Held-out perplexity, academic text / code | ≤ 0.20 |
| Held-out perplexity, worst domain | 0.01 |
| Average in-loop downstream benchmark BPB | 0.17 |
| **HELMET at 8K, pre-extension** | **0.32** |

"Short context metrics are surprisingly poor predictors of long-context behavior." Even evaluating the *same benchmark* at 8K before extension gives R² = 0.32 and "fail[s] to predict double-digit swings in HELMET performance downstream." The pre-extension models cluster tightly; the differences only open up after extension.

**Attention sinks invert the received wisdom.** In this pool, **the presence of attention sinks correlates with *better* long-context performance (R² = 0.38)**. Models without QK-norm develop higher attention entropy and stronger sinks — and score higher. The mechanism the paper proposes for QK-norm's cost is exactly this: "high entropy and attention sinks are positive indicators — and QK norm reduces both." Attention entropy measured at positions 1K/4K/16K/32K clusters by QK-norm status and by little else. Retrieval-head analysis is inconclusive: weak prefill correlation, and "very low retrieval head scores across models" during generation, which the authors partly attribute to these models being early-in-training and weak.

**Where Llama 3 lands.** Llama 3's configuration — no QK-norm, no SWA, 8K pretraining, but GQA-8 — is among the strongest in the design space precisely because it declines three of the four penalties. But the Figure 1 caption is explicit: **"(Q), (O), and (L) indicate the Qwen 3, Olmo 3, and Llama 3 architectures; none of these is optimal."** Several OlmPool configurations beat Llama 3 on long-context extensibility; the paper does not name them in the main text.

**Robustness checks.** §5.2: extending pretraining to **2T tokens**, "the relative performance of the two architectures remains fairly consistent over the course of pretraining" — the gap is not a 140B-token artifact. §5.3: swapping in **YaRN** two-stage extension, "the same relative ranking of architectures is recovered by both methods, and the gap between architectures is *larger* with the better context extension method" — a better extension recipe amplifies rather than masks the architectural gap.

## Where it breaks

- **Scale.** The pool is trained to 140B tokens; "most modern models are trained far longer." The 2T check exists but has few data points.
- **Model weakness confounds mechanism.** Retrieval-head analysis may be underpowered — "these models may be too weak to reliably identify retrieval heads."
- **Initializations cannot be fully synchronized.** Adding GQA or QK-norm changes the parameter set, so exact init matching is impossible; Appendix B argues the init effect is smaller than the architectural effect, but it is an argument, not a control.
- **One extension recipe is primary.** RoPE-θ + 10B-token anneal; YaRN is a secondary check, at lower absolute scores.
- **Transformer-family only.** No Mamba/RWKV/SSM variants.
- **Mechanism is partial.** The authors say plainly that "there is more to understand mechanistically about the differences between OlmPool models."
- Per-configuration benchmark tables are sparse in the main text; most results are reported as figures, aggregates, and paired deltas.

## Relationship to existing wiki pages

- **[[grouped-query-attention]] — this is a qualification, not a contradiction.** Nothing here disputes GQA's serving economics (the existing page's claim, from Ainslie et al., is about decode-time memory bandwidth and KV-cache size, and that stands). What this paper adds is a *previously unpriced cost on a different axis*: GQA-8 vs. MHA-32 costs **−3.4 HELMET points** at 32K after context extension, and **−9 points when combined with SWA**. So GQA is not an unambiguous win — it is a trade whose downside shows up only in long-context downstream evaluation, after extension, and is invisible in loss. The existing page should be qualified, not corrected.
- **[[long-context-llm]] / [[length-generalization]] — direct evidence** that extensibility is set at pretraining time by choices unrelated to context, and that short-context validation cannot screen for it.
- **[[layer-normalization]]** — QK-norm, treated on that page (if at all) as a stability aid, carries the single largest measured penalty here (−3.8 to −6 points).

## Related

- [[long-context-llm]] · [[length-generalization]] · [[grouped-query-attention]] · [[multi-head-attention]] · [[attention-mechanism]] · [[layer-normalization]] · [[positional-encoding]] · [[transformer-architecture]] · [[pre-training]] · [[kv-cache]]
- [[llama]] — the Llama 3 design is one of the strongest configurations tested, but explicitly "not optimal"
- [[scaling-laws]] — pretraining is Chinchilla-optimal at 140B tokens for 7B params; §5.2 extends to 2T
- [[graham-neubig]] · [[cmu]] · [[university-of-washington]]

## References

- [arXiv:2608.10296](https://arxiv.org/abs/2608.10296)

## Related 2026 sources

- [[rope-scaling]] — the operation whose latent costs this paper prices
- [[qwen3.8-flash-next]] — a shipped model with the exact feature profile this paper warns about, and no published evaluation on its extended path
