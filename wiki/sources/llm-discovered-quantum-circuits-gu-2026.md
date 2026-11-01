---
title: "Discovering State-of-the-Art Quantum Circuits with Large Language Models"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - llm-discovered-quantum-circuits-gu-2026.md
arxiv_id: "2610.22367"
authors:
  - Andi Gu
  - Alek H. Kemeny
  - Susanne F. Yelin
  - Mikhail D. Lukin
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Discovering State-of-the-Art Quantum Circuits with Large Language Models

**Andi Gu, Alek H. Kemeny, Susanne F. Yelin, Mikhail D. Lukin**. Affiliations: Harvard Quantum Initiative and Harvard Physics; the [[anthropic|Anthropic]] STEM Fellows Program (Gu); Anthropic (Kemeny); QuEra Computing (Lukin). [arXiv:2610.22367](https://arxiv.org/abs/2610.22367), v1, quant-ph, 15 Oct 2026.

## Summary

This is evaluator-guided LLM evolutionary search applied to **reversible circuits that compute classical functions**, the arithmetic and decoding primitives that dominate fault-tolerant algorithm costs. Correctness for such circuits only needs checking on computational-basis inputs, so a candidate can be simulated as a bit string plus a phase rather than 2ⁿ amplitudes. That makes a cheap, exact evaluator possible. Circuits are ranked by **TQ-cost** (Toffoli count × peak qubits).

Candidates are **Python circuit-generating programs**, not gate lists, so one conceptual change can restructure the whole circuit. The search uses **OpenEvolve driven by Claude Mythos 5**:
- It starts from a hand-written spec and a blank program.
- An **ideation stage** proposes distinct algorithm families (for Reed–Solomon decoding: extended Euclidean, Berlekamp–Massey and PGZ). Each family evolves separately, and leaders are periodically cross-pollinated between families.
- Every iteration is a fresh agent session with a parent program, archive samples and short notes. The full history is never in context.

## Key points

- **Eight tasks** spanning DQI decoding, HHL rotation angle, Coulomb potential, fixed-point Gaussian, Fe₂S₂ alias-Select QROM, non-abelian lattice-gauge simulation and prime-field modular inversion. In **7 of 8**, the discovered circuit uses **both fewer Toffolis and fewer qubits** than the baseline.
- **Reed–Solomon DQI decoder:** **5.1M → 440k Toffolis** and 1,938 → 1,837 qubits, a **12×** TQ gain.
- **TQ gains against literature baselines** are 4.1× (Fe₂S₂), 7.7× (lattice gauge), 9.9× (Coulomb), 12× (Reed–Solomon) and **39×** (Gaussian). HHL improves from (1.6·10⁴, 300) to (1,348, 115).
- **Modular inversion (the eighth task).** The search found a new reversible architecture that makes **divsteps** practical: branch decisions are recovered from the current state during uncomputation instead of being stored. This improves the Toffoli–qubit trade-off. The paper reports no single headline gain because prior work trades Toffolis against qubits in different ways.
- **Three recurring design principles:** conditional routing (cheap controlled swaps instead of duplicated arithmetic); recomputing branch history during uncomputation; and accepting bounded-error failures on rare, costly inputs.

> [!warning] Newsletter vs paper
> "7/8 tasks beat best-known" is slightly loose. For the **Hermitian DQI decoder** the baseline is a *naïve reversible implementation*, because "no prior reversible compilation exists". The **4.1×–39×** range covers the tasks with literature baselines. Modular inversion is the eighth task, with no single gain reported. The other numbers (5.1M → 440k Toffolis, divsteps) match.

## What it does not show

- **No cost or compute accounting per search**, and no ablation of the model, the ideation stage or cross-pollination.
- **Correctness tolerances are task-specific.** Several circuits are bounded-error, with failure budgets of ≤10⁻³ or ≤2·10⁻⁴, so the gains partly trade exactness for resources.
- Resources are logical Toffoli and qubit counts. There is no layout or fault-tolerance overhead and no hardware run.

## Concepts & entities

- [[quality-diversity]]: separate algorithm-family islands with periodic migration.
- [[verifier]] · [[code-generation]]: an exact programmatic evaluator as the selection signal.
- [[anthropic]] · [[openmythos]]: the Mythos model line.
- Related evolutionary-coding sources: [[agentic-hardware-design-repository-level-code-evolution-yu-2026]], [[scaling-discovery-requires-forgetting-tan-2026]] (fresh sessions per iteration, as here).

## References

- [arXiv:2610.22367](https://arxiv.org/abs/2610.22367)
