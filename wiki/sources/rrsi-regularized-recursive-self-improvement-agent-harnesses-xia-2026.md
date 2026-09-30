---
title: "RRSI: Regularized Recursive Self-Improvement of Agent Harnesses"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - rrsi-regularized-recursive-self-improvement-agent-harnesses-xia-2026.md
arxiv_id: "2609.24972"
authors:
  - Peng Xia
  - Rujun Han
  - Zifeng Wang
  - Yanfei Chen
  - Yufan Zhuang
  - Yoonho Lee
  - Chengsong Huang
  - Han Yu
  - Zhongying CuiZhu
  - Yifei Ming
  - Huaxiu Yao
  - Burak Gokturk
  - Tomas Pfister
  - Chen-Yu Lee
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# RRSI: Regularized Recursive Self-Improvement of Agent Harnesses

**Peng Xia, Rujun Han, … Tomas Pfister, Chen-Yu Lee** (Google Cloud AI Research; UNC-Chapel Hill; [[stanford-university|Stanford]]; WashU) — [arXiv:2609.24972](https://arxiv.org/abs/2609.24972), v2, cs.LG, 23 Sep 2026. Preprint; no venue. Code: `github.com/google-research/rrsi`.

## Summary

Harness evolution reuses one finite evolve set across many rounds, so it is **adaptive empirical optimization**: it can fit benchmark-specific patterns, promote winners of evaluation noise, and pile on complexity. RRSI keeps the harness edit space fully open and instead regularizes the *search trajectory*, borrowing classical regularizers by analogy. On the **proposal side**: a cosine-annealed cap on how many edits a candidate may bundle (L0-style; 4 → 1 edits over 20 rounds), a history of every tested hypothesis and its outcome, and a stall rule that reserves proposals for untouched components. On the **selection side**: a critic rejects diffs that encode task names, answers or inert machinery *before* they are scored; a noise-adjusted floor (δ estimated by re-running the unchanged base harness) blocks downhill drift; token growth must be paid for by gain (Ridge-style); and components with no recent positive contribution become deletion targets (Lasso-style).

The trade is explicit: **RRSI posts the smallest evolve-set gain of any evolved harness and the best out-of-distribution result.**

## Key points

- **Setup.** Policy, proposer, analyst and leakage critic are all Claude Opus 4.8. Three domains, each evolved on one suite and run unchanged elsewhere: coding (evolve [[terminal-bench|Terminal-Bench]] 2.1 → [[swe-bench-verified|SWE-bench Verified]]), agentic workspace (Harvey LAB 120 evolve / 40 held-out → JobBench, GDPval, APEX-Agents), engineering (EngDesign → Frontier-Eng, both simulator-graded, so no judge). T = 20 rounds and k = 2 trials per task (engineering: 40 and 4).
- **Headline gains over H₀.** Evolve: +6.0 Terminal-Bench, +4.9 EngDesign, +1.1 Harvey. Held-out: SWE-bench Verified +1.8, Harvey ID +2.3, JobBench/GDPval/APEX +4.7/+3.5/+3.7, Frontier-Eng +4.3 Medal points. No held-out split regresses.
- **Against four prior methods** (Table 1, agentic workspace; same H₀, evolve set and candidate budget):

| Method | Harvey evolve | Harvey ID | OOD avg (Job/GDP/APEX) |
|---|---|---|---|
| H₀ | 89.4 | 86.9 | 39.7 |
| [[meta-harness-lee-2026\|Meta-Harness]] | **93.0** | **89.2** | 40.6 |
| [[agentic-harness-engineering-lin-2026\|AHE]] | 90.7 | 88.7 | 39.2 |
| TTHE | 91.1 | 88.5 | 38.0 |
| HarnessX | 91.8 | 89.1 | 39.7 |
| **RRSI** | 90.5 | **89.2** | **43.6** |

  The ranking inverts out of distribution: the best evolve-set method adds under a point OOD, and AHE and TTHE end below the harness they started from.
- **Ablation** (Table 2): stripping both regularizer groups gives the *highest* evolve score (92.8) and OOD 40.3, within a point of H₀, at **3.80M tokens per trial vs 2.42M** for RRSI. Dropping only the acceptance rules costs 2.6 OOD points; dropping only the proposal rules costs 1.7.
- **Backbone transfer.** Evolved with Gemini 3.5 Flash: Terminal-Bench 64.6 → 78.7 (+14.1, the abstract's "up to 14.1"), SWE-bench Verified +2.2. The same harness under the unseen Gemini 3.1 Flash Lite: 11.2 → 14.6.
- **Cost.** AHE's final harness uses 3.82M tokens per trial (58% more than RRSI) for 4.4 fewer OOD points. **No evolved harness is as cheap as H₀** (1.56M tokens, 21.2 steps vs RRSI's 26.3), so part of every gain, RRSI's included, is bought with test-time compute.

> [!note] Abstract vs body
> The abstract's "30% fewer policy tokens than the unregularized evolution" does not match Table 2: 2.42M vs 3.80M is **36% fewer**. The intro's "outperforming the average prior baseline by up to 22.9%" is not traced to any table.

## What it does not show

- **No matched-compute search baseline.** All comparators are other harness-evolution methods at an equal *candidate* budget. There is no parallel-sampling or best-of-k arm at RRSI's 2.42M tokens per trial, which is the control [[rethinking-evaluation-harness-evolution-wang-2026]] demands. What RRSI does answer is that paper's *transfer* objection: its held-out gains are real and larger than prior methods', even though they are modest (+1.8 to +4.7).
- Single evolution run per arm, with no seeds or intervals. The noise band δ is about 3 passes out of 178 trials on coding, similar in size to several reported deltas.

## Concepts & entities

- [[agentic-harness-engineering]] — the method family, and the four baselines it beats out of distribution.
- [[self-improving-agent]] — frames harness evolution as recursive self-improvement at the agent-system level.
- [[harness-vs-model-attribution]] — the backbone-transfer tables are a harness-held-fixed, model-varied cut.
- [[llm-as-a-judge]] — three of the workspace benchmarks are judge-graded. The engineering domain exists to rule judge-gaming out.
- [[benchmark-contamination]] — the leakage critic is a pre-scoring contamination screen for harness edits.
- [[harness-continual-learning-kang-2026]] — a parallel guarded-commit gate; both find unrestricted acceptance ends worst.
- [[evo-bench-can-language-models-improve-agent-harness-huang-2026]] · [[prime-agent-self-improving-rlm-harness-karten-2026]] — cited prior work on the evolve-to-transfer gap.

## References

- [arXiv:2609.24972](https://arxiv.org/abs/2609.24972)
- Baselines: Meta-Harness (Lee et al.), AHE (Lin et al., arXiv:2604.25850), TTHE (Nie et al.), HarnessX (Chen et al., arXiv:2606.14249).
