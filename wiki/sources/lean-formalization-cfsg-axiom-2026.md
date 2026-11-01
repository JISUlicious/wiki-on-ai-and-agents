---
title: "Lean Formalization of the Classification of Finite Simple Groups"
type: source
created: 2026-11-01
updated: 2026-11-01
sources:
  - lean-formalization-cfsg-axiom-2026.md
authors:
  - Dean Cureton
  - Vasily Ilin
  - Bhavik Mehta
  - Niels Voss
year: 2026
status: complete
importance: high
tags:
  - 2026
  - blog-post
---

# Lean Formalization of the Classification of Finite Simple Groups

**[[axiom-math|Axiom]]**, 26 Oct 2026. [Blog post](https://axiommath.ai/research/the-enormous-theorem-formalized/), [Lean repository](https://github.com/AxiomMath/CFSG) (Apache-2.0) and [proof map](https://cfsg.axiommath.ai/). The authors above are the core contributors; the citation lists 31 people. Surfaced by the 31 Oct AI Agents Weekly.

## Summary

Axiom released what it describes as the first complete computer-checked proof of the Classification of Finite Simple Groups, the "Enormous Theorem", whose original proof runs to about 15,000 pages across more than 500 articles. A [[multi-agent]] swarm directed by engineers and mathematicians wrote it in about 24 days of project time, following the first-generation proof. The README describes it as sorry-free Lean 4, on Mathlib and TauCeti, using only the three standard axioms and no `native_decide`.

## Key points

- **Scale.** 43.1M lines of Lean, 2.05M declarations and 715 papers and books. The repository builds in under 24 hours with some unreleased Lake changes and optimizations.
- **How the swarm worked.** A shared proof map held the dependencies. Agents claimed milestones through GitHub issues, reported in Slack, and delegated missing prerequisites. Lead agents decomposed arguments and reviewed statements, and others searched or wrote proofs, using a changing mix of models.
- **Humans mattered.** On 13 Oct only one milestone closed, and engineers ran a replanning effort. Early on, agents had **assumed large prerequisites instead of proving them**, so tens of thousands of lines were deleted. People also check that the formal statement says what is intended, since Lean checks only the deductions.
- **Findings in the literature.** Axiom logs **34 noteworthy deviations** from the published record. Agents found explicit counterexamples to a closure assertion in Walter's 1986 route to the Unbalanced Group Theorem, and Axiom replaced it with a new argument (Theorem 2.3, "preservation of simple tops"). Another was an equality in the Aschbacher–Smith quasithin volumes that fails for SL₂(5).
- **Expert involvement.** Richard Lyons and Ronald Solomon were consulted and sit on an advisory board for the errata.

> [!warning] Figures differ between Axiom's own pages
> The project site's headline stat says 100 milestones, while its plan panel says 108, and coverage of the post repeats 108. Coverage also gives the date as both 26 and 27 Oct. These are minor, but the milestone count should not be quoted as exact.

## What it does not show

- **New mathematics at the level of the classification.** It formalizes a known theorem, so its value is certainty and the errata. It is not a new classification, and the Walter fix covers the simple-group form that CFSG needs.
- **Autonomy.** It was human-directed, with replanning and expert consultation. Axiom's post gives no exact agent count or compute cost.
- **Independent audit.** Lean's kernel checks the deductions and anyone can run the build, but no third-party review of the statement, the Lean development or the 34 errata was found at ingest time. The paper on the fix is on Hexagon (2610.00575), and its peer-review status was not checked.
- **Stock-toolchain reproducibility, yet.** The sub-24-hour build relies on changes Axiom says it plans to upstream.

## Concepts & entities

- [[axiom-math]] · [[multi-agent]]
- [[exponential-time-hypothesis-is-false-openai-2026]]: a claimed new result from the same week, in contrast to this formalization of a known one.
- [[leap-formal-mathematics-agentic-kung-2026]] · [[numina-lean-agent-liu-2026]]: agentic Lean proving at a much smaller scale.

## References

- [Axiom blog post](https://axiommath.ai/research/the-enormous-theorem-formalized/)
- [AxiomMath/CFSG repository](https://github.com/AxiomMath/CFSG)
- [Proof map](https://cfsg.axiommath.ai/)
