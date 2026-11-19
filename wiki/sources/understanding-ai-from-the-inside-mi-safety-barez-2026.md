---
title: "Understanding AI from the Inside: The State of Mechanistic Interpretability for AI Safety"
type: source
created: 2026-11-19
updated: 2026-11-19
sources:
  - understanding-ai-from-the-inside-mi-safety-barez-2026.md
arxiv_id: "2611.08438"
venue: arXiv preprint (cs.CL)
authors:
  - Fazl Barez
  - Tung-Yu Wu
  - Hunar Batra
  - Gabriele Dominici
  - Florent Draye
  - Yanai Elazar
  - Damiano Fornasiere
  - Antía García
  - Abir Harrasse
  - Riley Harris
  - Sara Hooker
  - Michael Lan
  - Belinda Z. Li
  - Aaron Maiwald
  - Narmeen Fatimah Oozeer
  - Naomi Saphra
  - Or David Shafran
  - Adi Simhi
  - Robert Trager
  - Shriyash Kaustubh Upadhyay
  - Ziyu Yao
  - Yu Zhao
  - Shay B. Cohen
  - Mor Geva
  - [[yoshua-bengio]]
  - Stella Biderman
  - Hassan Sajjad
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - survey
---

# Understanding AI from the Inside (Barez, Wu et al., 2026)

**Fazl Barez and Tung-Yu Wu** (core contributors; Oxford) with 20 co-authors and senior advisors Shay B. Cohen, Mor Geva, [[yoshua-bengio|Yoshua Bengio]], Stella Biderman and Hassan Sajjad. Affiliations include Oxford, Martian, MIT, [[anthropic|Anthropic]], Mila, LawZero and [[eleutherai|EleutherAI]]. [arXiv:2611.08438](https://arxiv.org/abs/2611.08438), cs.CL, v2 9 Nov 2026. 183 pages including references and appendices. Code notebooks reproduce key findings.

## Summary

A long survey of [[mechanistic-interpretability]] (MI) organised around one question: **does understanding how models work actually make them safer?** The authors' answer is cautious. They say the assumption that mechanistic understanding implies safer systems is "largely untested", and that MI should be one layer of a defence-in-depth strategy, not a standalone guarantee.

The survey is a reference work. It reports no new experiments. Its two own contributions are a five-dimension evaluation framework and a classification of the field into six research orientations.

## Key points

- **Why behaviour alone is not enough** (Sec. 2). Three problems: test-set contamination and saturation; benchmark gaming, where two models with the same outputs use different mechanisms; and evaluation awareness, where a model behaves safely only when it detects a test (sleeper agents, Hubinger et al. 2024).
- **Foundations** (Sec. 4–5). Features, [[polysemanticity]], transformer computation via the [[residual-stream]], [[circuits]]. Four core hypotheses: [[linear-representation-hypothesis|linear representation]], [[superposition]], privileged access, and universality / Platonic representation.
- **Methods covered** (Sec. 6–11). [[sparse-autoencoder|SAEs]] and their failure modes (dead features, duplication, absorption and splitting, instability across seeds), [[transcoder|transcoders]] and crosscoders, interpretability-by-design (weight-sparse training, KANs), circuit discovery and causal verification, [[logit-lens]], tuned lens, Patchscopes, LatentQA, [[probing]], model diffing, knowledge localisation, [[steering]], learning dynamics, and training-data attribution.
- **Five evaluation dimensions** (Sec. 7.5): attribution faithfulness, completeness, stability, causal validity, human interpretability. "No single dimension suffices." Metrics are defined from intervention effects on model components.
- **Six research orientations** (Sec. 12): pragmatic, ambitious, developmental, across time, introspective, automated. Example for pragmatic: a [[steering-vectors|steering vector]] that suppresses Claude Sonnet 4.5's evaluation awareness, useful without a full explanation of the mechanism.
- **Safety applications** (Sec. 13): concept erasure, knowledge editing, steering, mechanistic monitoring, automated interpretability agents. A table maps each application (e.g. monitoring for lying or [[sycophancy]]) to the methods the authors find most promising.
- **Risks of MI itself** (Sec. 13.7): false confidence from partial understanding, failure under deployment, misuse by attackers, and opportunity cost.
- **Open problems** (Sec. 14). Evaluation methodology and safety benchmarks are named "the field's most pressing open problems". Most tools were built for transformers; whether they transfer to other architectures is open.

## What it does not show

- No new empirical results. Claims about method quality summarise cited work.
- The central question is framed, not answered. The paper argues that MI's safety value should be tested against non-interpretability baselines, but does not run that comparison (inference).

## Concepts & entities

- [[mechanistic-interpretability]] · [[ai-safety]] · [[circuits]] · [[superposition]] · [[polysemanticity]] · [[monosemanticity]]
- [[sparse-autoencoder]] · [[transcoder]] · [[probing]] · [[logit-lens]] · [[steering]] · [[steering-vectors]] · [[activation-engineering]]
- [[linear-representation-hypothesis]] · [[residual-stream]] · [[grokking]] · [[reward-hacking]] · [[sycophancy]]
- [[yoshua-bengio]] · [[anthropic]] · [[eleutherai]]

## References

- [arXiv:2611.08438](https://arxiv.org/abs/2611.08438)
- Sparse autoencoders: Bricken et al. 2023; Cunningham et al. 2024 ([[sparse-autoencoders-cunningham-2023]]). Circuit tracing: Ameisen et al. 2025; Lindsey et al. 2025.
- Sleeper agents: Hubinger et al. 2024. Pragmatic interpretability: Nanda et al. 2025. Ambitious interpretability: Gao 2025.
