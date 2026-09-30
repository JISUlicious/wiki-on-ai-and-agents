---
title: "ScientistTwo: Pioneering the Human Knowledge Frontier with Autonomous AI"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - scientisttwo-autonomous-ai-human-knowledge-frontier-nam-2026.md
arxiv_id: "2609.19644"
authors:
  - Jaehyun Nam
  - Jinsung Yoon
  - Yanzhou Pan
  - Yubo Wang
  - Rui Meng
  - Parthasarathy Ranganathan
  - Tomas Pfister
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# ScientistTwo: Pioneering the Human Knowledge Frontier with Autonomous AI

**Jaehyun Nam, Jinsung Yoon, Yanzhou Pan, Yubo Wang, Rui Meng, Parthasarathy Ranganathan, Tomas Pfister** (Google Cloud AI Research; University of Waterloo) — [arXiv:2609.19644](https://arxiv.org/abs/2609.19644), v1, cs.AI, 17 Sep 2026. Preprint; no venue.

## Summary

An end-to-end autonomous research pipeline and the successor to ScientistOne (Meng et al. 2026). Its input is an accepted paper's problem and codebase. Its output is a new method, a manuscript and a runnable repository. Every stage runs the same *candidate → critic → accept / refine / discard* loop. The stages are: limitation extraction and novelty-checked seed ideas; **subset-first evaluation**, which screens each idea on benchmark slices before scaling to the full multi-dataset suite; idea evolution from experimental traces; automated ablation planning; manuscript drafting; **simulated peer review with a rebuttal agent** that runs supplementary experiments; and a meta-reviewer that can send the *idea itself* back for revision. Coding is delegated to [[claude-code|Claude Code]] (Opus 4.8), and reasoning runs on Gemini 3.6 Flash. A mean run takes **2.5 days and ~$3,765**.

## Key points

- **Task pool.** 107 problems drawn from 38 NeurIPS 2025, 5 ICLR 2026 and 64 ICML 2026 spotlight papers. ScientistTwo produced an improved result on **86/107 (80.4%)**. The **mean relative gain of 25.2% (median 7.7%) is computed over those 86 successes only**, from gains parsed out of the generated tables by Gemini 3.6 Flash (10 parses, averaged).
- **Automated review** (Table 3). ScholarPeer is also used inside the pipeline to refine drafts, so it is in-distribution. The Stanford Agentic Reviewer (SAR) is held out.

| Papers | ScholarPeer rating / accept | SAR rating / accept |
|---|---|---|
| NeurIPS 2025 accepted (human) | 6.2 / 65.8% | 5.5 / 76.3% |
| ICML 2026 spotlight (human) | 6.9 / 79.7% | 6.1 / **96.9%** |
| ScientistTwo, overall (86) | **7.5 / 91.9%** | 5.7 / 72.1% |
| ScientistOne (21 public papers) | 3.8 / 14.3% | 4.1 / 0.0% |

  On the held-out reviewer, ScientistTwo sits between accepted NeurIPS papers and ICML spotlights. The baseline agents are scored on their own 2–21 *publicly released* papers, not on the same problems.
- **The review loop over-fits its own reviewer** (Table 5, 49 ICML problems). From review round 1 to round 2, ScholarPeer acceptance rises 79.6% → 93.9% while **SAR acceptance falls 73.5% → 69.4%** (rating 5.8 → 5.7).
- **Against AutoSOTA** (Table 4): mean gain 25.2% vs 7.5% overall, but AutoSOTA leads on the ICLR 2026 subset (7.2% vs 3.8% mean) and succeeds on more tasks (105 vs 86). AutoSOTA is the system whose public logs [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]] mined.
- **Integrity** (Table 7, CoE audit): all four checks pass with the refinement agents on. Without them, 1/50 runs has a specification violation, 19 of ~1,800 references are hallucinated, and 11 of 49–50 papers show method–code mismatch. One otherwise-successful run contained **reward hacking** and was filtered out (footnote 2).
- **Human review.** 9 reviewers on 33 NeurIPS-derived papers rate them 3.7 out of 5 overall standalone, and at **parity (3.0)** against the human originals. The human papers are preferred on method rigor (2.9).
- Iterating on a BPE-tokenization problem compounds: +10.9%, then +9.6%, then +8.2% over the previous best across three rounds.

## What it does not show

- **No matched-compute baseline.** There is no comparison with, for example, AutoSOTA or a plain best-of-N idea search at the same $3.8k per task. Every headline number is judged by an LLM reviewer (one of which the system trained against) or self-reported in generated tables. Nothing here is peer review of the output.
- The 21 failures are excluded from the gain and review statistics. Coding-backend generalization rests on n = 5 tasks.

## Concepts & entities

- [[multi-agent]] — a specialized-role pipeline with critic gates at every stage.
- [[llm-as-a-judge]] · [[agent-as-a-judge]] — the whole evaluation stack. The in-distribution vs held-out reviewer split (Table 5) is a clean example of judge over-fitting.
- [[reward-hacking]] — observed once and filtered by a post-hoc compliance agent.
- [[self-improving-agent]] — only in the loose sense of iterative frontier expansion. Weights and pipeline stay fixed.
- [[training-ai-scientists-replicate-research-falck-2026]] · [[autolab-long-horizon-auto-research-xu-2026]] · [[naturebench-coding-agents-nature-family-papers-wang-2026]] — adjacent paper-grounded research-agent evaluations.
- [[claude-code]] · [[mle-bench]].

## References

- [arXiv:2609.19644](https://arxiv.org/abs/2609.19644)
- Project page: scientist-two.github.io. Predecessor: ScientistOne (Meng et al. 2026). Comparator: AutoSOTA (Li et al. 2026).
