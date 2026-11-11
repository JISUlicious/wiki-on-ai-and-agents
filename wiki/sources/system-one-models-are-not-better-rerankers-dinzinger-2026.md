---
title: "System One Models Are Not Better Rerankers"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - system-one-models-are-not-better-rerankers-dinzinger-2026.md
arxiv_id: "2610.32318"
authors:
  - Michael Dinzinger
  - Kanishka Ghosh Dastidar
  - Michael Granitzer
year: 2026
venue: Preprint (arXiv cs.CL)
status: complete
importance: high
tags:
  - 2026
  - paper
---

# System One Models Are Not Better Rerankers

**Source:** Michael Dinzinger, Kanishka Ghosh Dastidar, Michael Granitzer (University of Passau; Interdisciplinary Transformation University Austria, IT:U) — [arXiv:2610.32318](https://arxiv.org/abs/2610.32318), cs.CL, v1 23 Oct 2026. Preprint.

## Summary

Within weeks of [[jev|Jev]]'s release, decision models were reported to be competitive rerankers. Those reports compared Jev with rerankers on other, mostly smaller backbones. This paper runs the controlled test: on the **same [[qwen|Qwen]] backbone** (six sizes, 0.6B to 27B), compare the untuned model asked zero-shot, its **decision fine-tunes** ([[kev|Kev]], Cloudflare's Clef) and its **reranker fine-tunes** (Qwen3-Reranker, Prism, and the authors' own at 9B and 27B). All rerank the same [[bm25|BM25]] top-100 on TREC DL 2019–2023 and five BEIR collections. A decision model is asked whether the passage answers the query and candidates are sorted by P(yes), with one passage per state (isolated) or all 100 in one state (joint).

## Key points

- **Reranker fine-tuning always wins or ties.** From 0.6B to 9B it beats decision fine-tuning by 5–29 nDCG@10 on every backbone, including zero-shot collections. At 27B the two tie.
- **Decision fine-tuning adds nothing up to 9B** over simply asking the untuned model. Kev-27B is the only decision fine-tune that beats its base. It is also a full fine-tune of the chat model trained on longer states, while the smaller Kevs are LoRA fine-tunes. Clef on the same 27B adds nothing, so size alone is not the reason.
- **Calibration depends on one sentence of prompt.** With the authors' question, decision models are better calibrated than published reranker fine-tunes from 4B up. With S1Rank's slightly different wording almost every model is worse calibrated, often by two to three times (Jev joint ECE 0.224 vs 0.075). Decision models then at best match the best reranker fine-tune (Kev-8B 0.243 vs Qwen3-Reranker-8B 0.257; Kev-27B 0.156 vs the authors' 27B 0.110). Untuned models gain just as much from the better wording.
- **Calibration does not rescue a worse ranking.** Keeping every passage with p ≥ 0.5, a reranker fine-tune keeps the better set up to 9B (F1@.5 51.0 for Prism-4B vs 42.6 for Kev-4B), even though it is usually worse calibrated. For decision models 0.5 is already near the best threshold (regret 0.9–3.1 points).
- **Why.** Reranker fine-tunes train on the target task with a ranking loss over hard negatives. Kev's data contain no MS MARCO or BEIR, and its general question-answering skill transfers to relevance about as well as base instruction following.
- **Framing.** A probability of relevance is "calibrated" only against a chosen definition of relevance and its base rate, which the decision-model user must write into the question. The authors' bottom line: "If you need a reranker, fine-tune a reranker."

## Limitations

The comparison uses released models, whose training data differ as well as their objective, so it tests decision fine-tuning as available today, not TypeSafe's unpublished RLCD. The authors' own reranker fine-tunes were trained once without hyperparameter search. Calibration was measured only on TREC DL at one relevance grade and threshold, and only one alternative wording was tried. All decision models were at most a month old.

## Concepts & entities

- [[system-one-model]] · [[jev]] · [[kev]] · [[typesafe-ai]] · [[logit-scoring]] — the models under test.
- [[calibration]] — the prompt-wording sensitivity is the main calibration finding.
- [[dense-retrieval]] · [[bm25]] · [[retrieval-augmented-generation]] · [[fine-tuning]] · [[lora]] — reranking setup and fine-tuning recipes.
- [[dual-process-theory]] · [[daniel-kahneman]] — origin of the "System One" name.
- Related: [[retriever-free-retrieval-decision-models-wan-2026]] (decision models used for navigation rather than reranking) · [[ollama-decision-models-2026]] (Clef).

## References

- [arXiv:2610.32318](https://arxiv.org/abs/2610.32318)
