---
title: "Discovering formally verified quantum programs with AI"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - formally-verified-quantum-programs-ai-shirobokov-2026.md
arxiv_id: "2611.09822"
venue: arXiv preprint (quant-ph)
authors:
  - Sergey Shirobokov
  - Ben Jaderberg
  - Katerina Hristova
  - George Tsoukalas
  - Johannes Bausch
  - Nicholas C. Rubin
  - Ryan Babbush
  - Swarat Chaudhuri
  - Pushmeet Kohli
  - et al. (25 authors)
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Discovering formally verified quantum programs with AI

**Source:** Sergey Shirobokov, Ben Jaderberg, Katerina Hristova, George Tsoukalas, Johannes Bausch, … Ryan Babbush, Swarat Chaudhuri, Pushmeet Kohli ([[deepmind|Google DeepMind]]; Google Quantum AI) — [arXiv:2611.09822](https://arxiv.org/abs/2611.09822), quant-ph, v1 6 Nov 2026 (v3 16 Nov 2026). Preprint.

## Summary

Quantum algorithms are hard to search over with AI because a classical computer can only test them on small or restricted instances. This paper replaces testing with proof. The authors build a library in the Lean theorem prover, based on quantum Hoare logic, that lets a program carry machine-checked proofs of both correctness and resource cost. Researchers write a problem specification in Lean with the program, its cost bound and the proofs left as `sorry` placeholders. Teams of Gemini 4 agents fill them in.

The teams share results in two ways: a wiki that holds each problem's best verified solution, reusable lemmas and notes on dead ends, and scheduled "conferences" where they exchange unverified ideas and critiques. Agents can edit their own files, so they could weaken a theorem or add an axiom. An external verifier (the Lean Comparator) therefore checks every submission against the frozen specification. Researchers only need to review the specification.

## Key points

Cost is spacetime volume Q·T (logical qubits × T gates).

| Problem | Application | Baseline Q·T | This work Q·T | Reduction |
|---|---|---|---|---|
| 25-bit fixed-point arcsin | State preparation | > 1.08 × 10⁸ | 3.70 × 10⁵ | > 290× |
| α + hydrogen stopping power | Fusion | 5.42 × 10¹⁵ | 4.59 × 10¹⁴ | 11.8× |
| C₂₃H₂₀N₃O non-Born–Oppenheimer dynamics | Photochemistry | 4.98 × 10¹⁵ | 6.06 × 10¹⁴ | 8.22× |
| Cytochrome P450 low-lying energies | Drug metabolism | 1.09 × 10¹² | 4.37 × 10¹¹ | 2.49× |

- **Parallel Grover search.** Verified circuits beat the strongest published protocol by up to 1.22× (find one target) and up to 2.57× (find all targets). The biggest gain comes from a new mechanism: a superposition over sets of candidates, steered by a walk on the Johnson graph.
- **arcsin in two stages.** Agents first searched a classically simulable gate set (kickmix), which reached 4.15 × 10⁵. Lifting that restriction under Lean proofs cut a further 11%. Lean is what makes the second stage checkable.
- **The improvements are interpretable.** Stopping power: moving electrons to a real-space grid lets the circuit exploit Pauli exclusion, cutting walk steps 12.8× at 1.42× the T gates per step. P450: a one-hot term register accounts for over 95% of the per-step T-gate saving.
- **Cross-team collaboration mattered.** In the non-Born–Oppenheimer problem the record changed teams eight times, and the final circuit combines techniques from three teams. Across the three science problems, 61% of the cost reduction came from within-team ideas, 15% from the wiki and 24% from conferences.
- All large-scale runs used Gemini 4 Carbon models. Team architecture was left to each researcher. Only the specification, verifier and workflow were fixed.

## Why it matters

This is the "verify, don't test" version of AI-driven discovery. A formal [[verifier]] lets the search go to thousands of qubits, where no simulator can check the answer, and it closes the specification-gaming loophole that open-ended agent search otherwise invites. It is a contrast to [[llm-discovered-quantum-circuits-gu-2026]], whose search was limited to circuits that can be simulated.

## Concepts & entities

- [[verifier]] · [[reward-hacking]] — a frozen Lean specification plus an external checker as the reward signal.
- [[multi-agent]] · [[multi-agent-memory]] — agent teams sharing a wiki and conferences.
- [[llm-agent]] · [[deepmind]] · [[google]].
- Related sources: [[llm-discovered-quantum-circuits-gu-2026]] · [[leap-formal-mathematics-agentic-kung-2026]].

## References

- [arXiv:2611.09822](https://arxiv.org/abs/2611.09822)
