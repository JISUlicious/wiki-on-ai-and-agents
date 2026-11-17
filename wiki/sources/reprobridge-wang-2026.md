---
title: "ReproBridge: Building Reproducibility into ML Research Artifacts"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - reprobridge-wang-2026.md
arxiv_id: "2611.08452"
venue: arXiv preprint (cs.AI)
authors:
  - Yubo Wang
  - Rui Meng
  - Jaehyun Nam
  - Tomas Pfister
  - Jinsung Yoon
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# ReproBridge: Building Reproducibility into ML Research Artifacts

**Source:** Yubo Wang, Rui Meng, Jaehyun Nam, Tomas Pfister, Jinsung Yoon (University of Waterloo; [[google|Google]]) — [arXiv:2611.08452](https://arxiv.org/abs/2611.08452), cs.AI, v1 5 Nov 2026.

## Summary

When a coding agent reproduces an ML paper, success depends partly on the released code and partly on the agent's skill at filling gaps: fixing dependencies, finding the right config, repairing a loader, and working out how logs become the reported number. That recovered knowledge usually dies in a chat log. ReproBridge keeps it. It is a six-role agent workflow (Intake, Planner, Builder, Reproducer, Distiller, Packager, plus an independent Reviewer) that turns a paper and its repository into a **refined reproducible artifact** with four parts: a repaired codebase, environment setup, experiment launchers, and a reproduction guide. The guide records each ambiguity, the resolution chosen, the reason and the affected tasks.

To test whether this helps, the authors build **ReproBridge-Bench**. Fifty ICML 2026 Spotlight papers with code were sampled, and 36 kept. Their main-table results give 1,463 curated tasks, of which 1,029 tasks from 387 experiments have five human-verified reference runs compatible with the published score. A fresh agent then attempts each task from either raw code or the refined artifact. Success requires a value inside a Student-t prediction interval around the references and no violation found by a separate execution-integrity Auditor.

## Key points

- **Main result (Table 3).** Mean success rises from 76.4% to 92.6% across four models: Sonnet 4.6 72.3 → 91.6, Sonnet 5 76.5 → 93.3, Opus 5 89.2 → 94.7, Gemini 3.8 Flash 67.6 → 90.8. The weakest raw-code model gains most (+23.2 pp).
- **Models converge.** The cross-model spread shrinks from 21.6 to 3.9 pp, and every model is above 90% with refined artifacts.
- **Cheaper reproduction.** Reproducer API spend falls 16.8–23.7% (19.7% on average), for example Sonnet 5 from $1,726 to $1,317. Construction, audit and GPU costs are not included.
- **Ablations (Sonnet 5, 287 tasks).** Raw code 73.9%; reused ordinary outputs (code, scripts, logs, but no guide) 81.2%; full artifact 93.0%. Dropping one component costs: code 5.2 pp, environment 4.5, guide 3.8, launchers 3.1.
- **Transfer.** On CORE-Bench Hard (45 capsules), refined artifacts lift capsule success from 62.2% to 73.3% and cut cost per success from $8.43 to $4.34.
- **Matching numbers are not enough.** Against two humans checking 818 outcomes, the Codex Auditor reaches 88.6% precision and 79.5% recall on invalid outcomes, against 85.7% and 61.5% for Claude. In one case, a reported 206 s epoch time was a projection; the measured time was 32.5 s.
- Only 70.3% of curated tasks even had reference runs compatible with the paper. The rest were numerical mismatches (12.5%), blocked execution (12.7%) or compute/access limits (4.4%).

## Caveats

- The comparison bundles upstream investigation, repair, review and packaging together, so it does not isolate the value of reorganizing the same outputs.
- Numerical compatibility does not certify scientific validity, and refined artifacts can preserve wrong assumptions.

## Concepts & entities

- [[agent-evaluation]] · [[mle-bench]] · [[benchmark]] — agents doing ML research engineering, scored against verified references.
- [[code-generation]] · [[llm-agent]] — coding agents as reproducers.
- [[google]] · [[claude]].

## References

- [arXiv:2611.08452](https://arxiv.org/abs/2611.08452) · project page: https://wyyyb.github.io/reprobridge_website/
