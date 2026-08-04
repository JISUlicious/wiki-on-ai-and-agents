---
title: "Two-Level Meta-Rubrics for Evaluating Open-Ended Generation: GAMUT, a Benchmark for Factual Completeness"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - gamut-two-level-meta-rubrics-chen-2026.md
arxiv_id: "2607.19322"
year: 2026
authors:
  - Xilun Chen
  - Zhaleh Feizollahi
  - Ross Goodwin
  - Seungwhan Moon
  - Scott Yih
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Two-Level Meta-Rubrics for Evaluating Open-Ended Generation: GAMUT, a Benchmark for Factual Completeness

**Xilun Chen, Zhaleh Feizollahi, Ross Goodwin, Seungwhan Moon, Scott Yih, Pinar Donmez, Babak Damavandi, Luna Dong** (Meta AI) — [arXiv:2607.19322](https://arxiv.org/abs/2607.19322) (v1, 21 Jul 2026; paper dated 22 Jul 2026). No venue in the arXiv comments. Code: `github.com/facebookresearch/GAMUT`.

## Summary

Long-form factuality evaluation is dominated by **precision**: decompose the response into atomic claims, search, verify each (FActScore, VeriScore). That answers "is what it said true" and says nothing about "did it say what it should have said." This paper argues that **completeness is the missing half of factuality**, and that both the precision and the thin existing recall literature share a deeper flaw — they assume answer quality reduces to a flat list of independent boolean checks. Real knowledge-seeking answers break that assumption in three ways: **open sets** where coverage matters but no specific item is required (name enough dishes using this ingredient — two answers naming *different* valid dishes can be equally good), **ordered processes** where stating correct milestones out of sequence is an error, and **relationships** between facts that independent checks cannot express.

The fix is a **two-level rubric representation**, and this is the paper's actual contribution. Level one is a **structured meta-rubric**: items typed as *Simple Knowledge*, *Strict List* (every item required), *Flexible List* (a pool with a minimum-coverage threshold), *Process* (ordered steps), or *Relationship*, each carrying an importance tier — **Answer-Critical**, **Valuable**, or **Context** — plus optional *meta-insights* (a synthesis across items, like "it essentially always uses some red meat"), all cited to human-verified web snippets. Level two is a **flat binary checklist mechanically compiled from it by fixed deterministic rules**. A strict list becomes one check per item; a flexible list becomes a self-contained baseline check ("mention at least 2 of the following: …" with all options enumerated) at the list's tier, plus per-item or higher-coverage checks one tier *lower*; a process becomes presence checks per step plus an explicit **sequence check** over the required steps. The point of the split: rubrics are authored, verified and revised where structure is expressible, but graded where LLM judges are reliable — flat, independent, binary. The structure survives compilation in *which checks exist and how they are weighted*.

**GAMUT** (Grounded Assessment of Multimodal Factuality) instantiates this on "everyday deep research": naturally phrased questions a person would ask a wearable assistant about something in view, that nonetheless need multi-step web research. 1,813 questions, one per image, drawn from CRAG-MM wearable imagery (~80% egocentric smart-glasses photos, often blurred/occluded/low-light) across 10 domains. Questions and rubrics are both LLM-proposed (Gemini 3.1 Pro) and expert-human-verified over multiple rounds; a "stranger test" filter rejects leading questions that presuppose the entity is already identified. 14 frontier and open-weight models are evaluated. The best, **Gemini 3.1 Pro, scores 58.7%** — the benchmark is unsolved, the ranking is stable across three different judges, and the dominant failure mode is **omission, not error**.

## Key points

**Main results (Table 1), 1,813 multimodal questions, judged by Gemini 3.1 Pro.** GAMUT score range is [−1, 1], shown as %; AC/Val/Ctx are unweighted per-tier scores; verdicts are % of rubric elements.

| Model | GAMUT | AC | Val | Ctx | meets | partial | missing | contra. |
|---|---|---|---|---|---|---|---|---|
| Gemini 3.1 Pro | **58.7** | 63.9 | 51.9 | 45.1 | 55.2 | 11.5 | 29.2 | 4.1 |
| Gemini 3 Flash | 57.9 | 63.0 | 51.6 | 44.7 | 54.9 | 12.1 | 28.5 | 4.5 |
| Claude Opus 4.8 | 53.1 | 59.4 | 43.9 | 40.4 | 45.3 | 13.5 | 38.6 | **2.6** |
| Gemini 2.5 Pro | 51.8 | 55.8 | 47.2 | 39.7 | 52.1 | 11.5 | 30.5 | 5.9 |
| Claude Opus 4.6 | 50.5 | 54.6 | 45.7 | 38.8 | 49.2 | 12.9 | 32.6 | 5.2 |
| GPT-5.4 | 42.6 | 46.8 | 37.8 | 29.0 | 39.5 | 14.0 | 42.0 | 4.5 |
| Claude Sonnet 4.6 | 41.5 | 46.7 | 34.3 | 30.5 | 38.4 | 13.5 | 43.2 | 4.9 |
| GPT-4o | 34.2 | 39.1 | 28.0 | 21.5 | 29.7 | 12.6 | 54.3 | 3.4 |
| Qwen3-VL 235B | 33.0 | 34.1 | 32.6 | 26.4 | 39.4 | 12.1 | 39.8 | 8.7 |
| Llama 4 Maverick | 18.3 | 19.5 | 17.3 | 12.3 | 22.9 | 11.7 | 57.9 | 7.6 |
| Llama 4 Scout | 14.1 | 14.4 | 14.3 | 10.5 | 21.1 | 11.3 | 59.3 | 8.3 |
| Qwen3-VL 8B | 13.7 | 12.3 | 17.0 | 11.5 | 27.3 | 10.6 | 50.7 | 11.5 |
| Llama 3.2 90B | 11.8 | 12.7 | 10.9 | 8.2 | 17.8 | 10.0 | 65.1 | 7.2 |
| Llama 3.2 11B | 5.1 | 3.8 | 7.8 | 4.3 | 14.2 | 9.2 | 68.3 | 8.3 |

- **Omission dominates.** *missing* verdicts are ~two-thirds of rubric elements for the weakest models and still >25% for the strongest. Contradiction stays low for strong models (Claude Opus 4.8 lowest at 2.6%) and climbs for weak ones (Qwen3-VL 8B 11.5%). This is precisely the axis precision-based factuality evaluation cannot see.
- **Scoring formula.** Per-check verdicts are four-way — *meets* = 1, *partially meets* = λ, *missing* = 0, *contradicts* = µ — with **λ = 0.5, µ = −2**. The tier score is `(M + λP + µC) / (M + P + S + |µ|C)`, so a contradiction both subtracts credit *and* counts as more than one check: a cautious answer outranks a confidently wrong one. Tiers combine with global weights **w_AC = 0.6, w_Val = 0.3, w_Ctx = 0.1** (global, not per-question check counts, so many minor Context checks cannot drown out a few Answer-Critical ones); missing tiers redistribute their weight.
- **Judge robustness (Table 2).** All 14 models re-scored by Gemini 3.1 Pro, Claude Opus 4.8 and Qwen3-VL 235B on all 1,813 questions. Gemini and Claude give an **identical ranking**, per-model scores **within 1.8 points**, and **no self-preference** — each scores itself within 0.3 points of how the other scores it. The smaller open-weight judge (Qwen3-VL 235B) is uniformly **4–11 points more lenient** but preserves the broad ordering with only adjacent swaps.
- **Text-only variant (Table 3), 1,806 questions** (7 remain image-dependent after rewriting the subject explicitly; rubrics, judge and scoring unchanged). Every model gains, roughly uniformly: Gemini 3.1 Pro 58.7 → 74.1 (**+15.4**), Gemini 3 Flash 57.9 → 71.2 (+13.3), Claude Opus 4.8 53.1 → 60.8 (+7.8), GPT-4o 34.2 → 58.1 (**+23.9**), Llama 3.2 11B 5.1 → 22.7 (+17.6). Visual identification is a near-constant tax rather than the axis separating models, so the multimodal ranking reflects **knowledge completeness, not perception**.
- **Rubric statistics.** **15.2 binary checks per question on average** (median 15, max 41), split ~**5.9 Answer-Critical / 5.8 Valuable / 3.5 Context**. Each rubric draws on ~10 web snippets; across the benchmark the rubrics cite **over 9,400 distinct web pages**.
- **Structure is not incidental — the load-bearing justification for the whole framework: 98% of questions require at least one component beyond isolated facts** that a flat checklist cannot represent. Structural depth: flexible lists have a median of 4 options (12% are "long", >7 items); processes a median of 4 steps (57% mix required and optional); strict lists a median of 2 items; relationships a median of 2 entities (52% multi-entity); meta-insights appear in 13% of questions (295 total).
- **Question diversity.** Median 23 words; the 25 most common opening phrases cover under 60% of questions; ~1 in 6 do not begin with a standard interrogative. Domain sizes include Plants & Gardening (n=300), Local (n=290), Vehicle (n=258), Food (n=244), Everyday Objects (n=196).
- **Construction is LLM-proposed, human-verified at every stage.** Gemini 3.1 Pro generates candidate questions and rubrics; two annotators review each question independently with a third adjudicating; rubrics get an LLM self-refinement pass (re-verify every snippet by browsing its source, then re-research from scratch), then in-house human revision, then another LLM+human round. Annotators are shown the meta-rubric but edit the **binary** rubric directly, so they audit the compilation itself. 1,843 questions survive question creation; **1,813** have a sound verified rubric; 12 upstream CRAG-MM entity/ambiguity errors were hand-corrected.
- **The framework generalizes beyond factuality** — the authors present the two-level meta-rubric as a recipe for any rubric-based evaluation of open-ended generation where a flat list of binary criteria falls short.

## Concepts & entities

- [[llm-as-a-judge]] — the framework is a direct answer to judge unreliability: keep the judge on flat independent binary checks, and put all the structure in a layer the judge never sees.
- [[verifier]] — GAMUT's compiled checklist is a completeness-oriented verifier, complementary to the decompose-search-verify pipelines (FActScore, VeriScore) that verify precision.
- [[hallucination-detection]] — the four-way verdict scheme separates *missing* from *contradicts* and penalizes contradiction at µ = −2, so an omission and a confident falsehood no longer score the same.
- [[agent-evaluation]] — "everyday deep research" questions require multi-step web research, and the rubric-vs-holistic-judgment tension here is the same one that shows up in grading agent trajectories.

## References

- [arXiv:2607.19322](https://arxiv.org/abs/2607.19322)
