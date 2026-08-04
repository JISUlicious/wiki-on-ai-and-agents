---
title: "Structured Output Collapses Answer Diversity Across 44 Language Models"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - structured-output-collapses-diversity-parikh-2026.md
arxiv_id: "2607.18476"
year: 2026
authors:
  - Tapan Parikh
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Structured Output Collapses Answer Diversity Across 44 Language Models

**Tapan Parikh** (Cornell Tech) — [arXiv:2607.18476](https://arxiv.org/abs/2607.18476) (v1, 20 Jul 2026). 12 pages, 1 figure; no venue. Companion to *The One-Word Census* ([arXiv:2607.12796](https://arxiv.org/abs/2607.12796)) by the same author. Code, data, and an interactive explorer in the `modelun` repository.

## Summary

Software does not consume language models in prose — it asks for JSON. This paper asks whether the same model answering the same question gives the same answer when the reply is requested as a serialization. It re-runs a frozen instrument, the One-Word Census: **31 single-turn prompts** ("Name a tree. Reply with one word only." plus one unconstrained "Pick a word."), **44 models**, no system prompt, requested temperature 1.0, four samples per cell. The only manipulated variable is a format clause appended to the prompt — `Reply with JSON only, in the form {"word": "<your answer>"}.` **No schema enforcement, no [[constrained-decoding]], no token masking: only the request.**

The metric is **answer-choice surprisal**: an add-one-smoothed leave-one-out measure, in bits, of how unlikely a model's answers are under the pooled answers of the other 43 models, computed by exact match on normalized one-word replies — no embeddings, no LLM judge, no human annotation. Crucially surprisal is computed **within each format column** (JSON answers scored against the JSON pool), so a column's convergence is an internal property of that column and cannot be an artifact of comparing registers.

The field converges sharply. Field-mean surprisal falls from **1.80 to 1.58 bits** under JSON (**−0.22 per model, p = .0002**); on the unconstrained "Pick a word", *serendipity* rises from **41% to 64%** of the pool and distinct answers fall from **52 to 36**. The tax is **progressive**: the most distinctive models lose the most while the conformist floor is immobile. It is a **sharpener, not a re-indexer** — the plain-chat modal answer survives in **28 of 31** categories and under 4% of JSON answer-mass lands on words the plain census never produced. And it is **not a decoding artifact**: adding a strict `response_format` json_schema on top of the request compresses only **0.03 bits further**. The collapse lives in the model's response to the register, not in the sampler.

## Key points

- **Progressive tax, not uniform.** deepseek-v3.2, the census's one genuine explorer among frontier-lab models, falls **2.63 → 1.32 bits (−1.31)** — half its measured distinctiveness. Also: hermes-4 −0.91, gpt-4o-mini −0.92, gpt-4-turbo −0.87, gpt-5.6-sol −0.65. The floor barely moves: claude-opus-4.8 −0.16, grok-4.5 −0.07, claude-sonnet-5 −0.05. 31 of 44 models become more generic; the raw range is nearly unchanged (2.2 → 2.0 bits) because a register-invariant minority holds the top.
- **Not regression to the mean.** Split-half test-retest reliability of the census score is **r = 0.94**, so pure re-measurement predicts 0.05 bits of shrinkage for the most distinctive model against the **1.31 observed**; every individually significant compressor clears the re-measurement null by **3.7–10σ**.
- **Individually significant movers: exactly 6 of 44** at BH-FDR q = .10, **all compressions** — deepseek-v3.2 −1.31, gpt-4o-mini −0.92, gpt-4-turbo −0.87, gpt-5.6-sol −0.65, qwen3 −0.45, gemini-3.1-pro −0.31. (hermes-4 at −0.91 misses only because its within-model dispersion is the panel's largest.) The remaining ~38, including every positive Δ, are indistinguishable from zero and the author explicitly declines to rank them.
- **Sharpener, not re-indexer.** The three mode flips are all categories whose plain-chat mode was weakest: insect *ant* (43%) → *butterfly* (60%), board game *chess* → *monopoly*, dance *salsa* → *tango*.
- **It is not a hidden temperature cut.** Self-distinctness (distinct answers ÷ samples, within model and category) is nearly flat across all six columns — plain 0.42, JSON 0.39, XML 0.40, YAML 0.42, CSV 0.43, brackets 0.43 — while surprisal drops 0.22 bits. A serving-layer temperature cut would crater it. **The collapse is positional**: mass relocates onto the field's mode without the model sampling any less around it.
- **Defaults are register-indexed** (the strongest result, read from discrete behavior rather than noisy scores). The four-sample census flags 144 stable off-modal defaults; re-sampled at **n = 20** in both registers they are genuine (median per-sample probability 0.90). JSON significantly shifts **76 of 144 (53%)** (Fisher exact, BH q = .10, against a ~10% false-positive floor), and **29% revert outright to the field's modal answer**. It also *installs* defaults: 81% of four-of-four JSON-only answers survive re-sampling — **Claude Fable 5 answers *cerulean* for colour 0% of the time in chat and 100% in JSON (p ≈ 10⁻¹¹)**, and *carpenter* for occupation 0% → 90% (p ≈ 10⁻⁹); opus-4.8 acquires *phoenix*, sonnet-4.6 *gold*. Same default, opposite reaction: gpt-5.6-terra and fable both answer *mango* for fruit 20/20 in chat, but under JSON terra flips to *apple* (20/20) while fable holds *mango* (19/20). The right object is a **per-register defaults profile**, not one personality the register reveals or hides.
- **Register gradient — the mechanism evidence.** Field-mean Δ-surprisal, compliance-conditioned (≥90% wrapper production): **JSON −0.22, XML −0.23, YAML −0.09, CSV −0.09, brackets +0.13**. Sign-flip permutation (20,000 draws) over both exchangeable units: JSON and XML significant on both (per model p = .0002 and .002; per category entropy −0.20, p = .0006 and .0004); YAML and CSV null on both (p = .75/.46 by category, .80/.35 by model); **brackets significantly *loosens*** (+0.13 per model, p = .009). The two compressing formats are exactly the two models are trained to *answer in* — the registers of function calls and tool use — which weights the mechanism toward **tool-use post-training** rather than a generic property of serialized text. On the 34 models compliant in all five formats the gradient sharpens (JSON −0.27, XML −0.26, brackets +0.13).
- **But a corpus-register component survives.** On the unconstrained "Pick a word", *every* serialization concentrates the pool including the two null ones: *serendipity* at 41% (plain) → 52% (CSV), 59% (XML), 64% (JSON), 65% (YAML), while brackets holds at 41%.
- **Enforcement adds almost nothing.** On the 36 of 44 models whose providers support strict `response_format` json_schema (the other 8 return no valid output), scored within that subset's own pool: open chat 1.79 bits, request 1.56, enforcement 1.53. **The request does the −0.22-bit work; enforcement adds −0.03.** This cleanly separates the effect from the "constraint tax" / grammar-constrained-decoding literature, which masks tokens at the sampler.
- **Compliance must be controlled or it manufactures fake divergence.** granite and mythomax emit valid CSV ~1% of the time; scored naively granite shows a spurious **+1.49 bits** in CSV. Separately, a fully compliant model can carry a real positive Δ through **stranding** rather than divergence — llama-4-maverick is 100% JSON-compliant with the panel's largest positive delta (+0.56) yet has *below-median* self-divergence (JSD 0.20 vs the field's 0.27, 7 of 31 modal answers change): it holds still while the field collapses around it. A panel-free self-divergence check is what separates the two.
- **Hygiene**: 27,280 format cells, 91 (0.3%) unrecoverable nulls, concentrated in YAML (56), negligible in JSON (1); parse-plus-junk survival ≥99% per model per column; a malformed reply falls back to the census rule so broken JSON cannot smuggle in a spurious "novel" answer.
- **Stated limitations**: one snapshot through one channel (OpenRouter); temperature 1.0 is not honored uniformly across providers; surprisal is panel-relative so absolute bits are not portable; each format is probed with a **single clause wording** (though the gradient already rules out key-priming and fill-in framing, since all four serialization clauses share the same slot name and template, and clause length runs the wrong way — CSV has the wordiest clause and is null); tool-call framing remains unmeasured. The author also discloses that Claude Opus 4.8 and Fable 5 helped run the battery and draft the text, and are themselves subjects of the study.

## Why it matters

> [!note] The evaluated surface is not the deployed surface
> Models are ranked on the prose surface — leaderboards, vibe checks, human preference — while software consumes them through structured output, and that surface is measurably more collapsed. Every diversity number the census reported is, for the deployment tier that increasingly matters, an overstatement. The author is careful about where this bites: **it costs you where the task admits many acceptable answers** — synthetic surveys, recommendations, brainstorming, judgement, tool choice — and **costs nothing where structured output carries a single correct value** (extraction, classification, routing).

## Concepts & entities

- [[constrained-decoding]] — the explicit contrast: this effect appears with **no** grammar, token masking, or schema enforcement, and adding decoder enforcement buys only 0.03 further bits, so it is a distinct phenomenon from the "constraint tax" literature.
- [[llm-as-a-judge]] — judge pipelines read the JSON persona, not the chat persona; the paper also notes the mode-collapse literature's finding that LLM judges prefer modal outputs, which is why this instrument is deliberately judge-free.
- [[agent-evaluation]] — an agent consults the model through the serialization surface, so benchmark numbers collected in chat describe a model with wider tastes than the one actually deployed.

## References

- [arXiv:2607.18476](https://arxiv.org/abs/2607.18476)
- Companion: Tapan Parikh, *The One-Word Census: Answer-Choice Conformity across 44 Language Models* — [arXiv:2607.12796](https://arxiv.org/abs/2607.12796)
