---
title: "LEAP: Supercharging LLMs for Formal Mathematics with Agentic Frameworks"
type: source
created: 2026-06-08
updated: 2026-06-08
sources:
  - leap-formal-mathematics-agentic-kung-2026.md
arxiv_id: "2606.03303"
authors:
  - Po-Nien Kung
  - Linfeng Song
  - et al.
first_author: Po-Nien Kung
year: 2026
venue: Google Cloud AI Research / Google DeepMind (preprint)
tags: [2026]
status: complete
importance: medium
---

# LEAP: Supercharging LLMs for Formal Mathematics with Agentic Frameworks

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-05-31–06-07).

## Summary

LEAP (LLM-in-Lean Environment Agentic Prover), from [[google-research]] (Google Cloud AI Research and DeepMind), is an agentic scaffold that wraps a **general-purpose** foundation model to do formal theorem proving in Lean, rather than fine-tuning a specialized prover. It leans on the base model's informal reasoning, instruction-following, and iterative self-refinement: complex problems are decomposed into smaller units, and an informal blueprint is bridged to a formal proof through continuous interaction with the **Lean compiler** as a verifier. The compiler's checks guide proof decomposition and repair, so correctness is mechanically guaranteed rather than merely plausible.

Empirically LEAP is strong. On the 2025 Putnam Competition it **solves all 12 problems**, matching recent specialized-prover breakthroughs. The authors also introduce **Lean-IMO-Bench**, a benchmark of IMO-style problems formalized in Lean (short statements, highly non-routine multi-step proofs); on it LEAP lifts the one-shot formal solve rate of general LLMs from below 10% to **70%**, surpassing the 48% set by a specialized gold-medal-caliber IMO system. It also autonomously formalizes a verified proof for a subproblem in Knuth's Hamiltonian decomposition of even-order Cayley graphs, showing research-level utility.

## Key points

- Agentic scaffold around a **general** LLM (no specialized prover fine-tuning) for formal math in Lean.
- **Verifier-guided proof decomposition**: complex goals split into units, informal blueprints bridged to formal proofs via continuous Lean-compiler interaction.
- **Putnam 2025**: solves all 12 problems, matching specialized frontier provers.
- Introduces **Lean-IMO-Bench**; raises general-LLM one-shot formal solve rate from <10% to 70%, beating a 48% specialized IMO system.
- Demonstrates research utility: a Lean-verified proof for a subproblem in Knuth's Hamiltonian decomposition of even-order Cayley graphs.

## Concepts & entities

- [[google-research]] — Google Cloud AI Research / Google DeepMind
- [[code-as-harness]] — the Lean compiler as the verifier wrapping the base model
- [[chain-of-thought]] — informal blueprints / iterative self-refinement driving formalization
- [[agent-benchmark]] — Lean-IMO-Bench as a new formal-math benchmark
- [[numina-lean-agent-liu-2026]] — related Lean-agent / formal-prover line of work

## References

- [arXiv:2606.03303](https://arxiv.org/abs/2606.03303) — Kung, Song, et al. (Google), 2026
- Source text: `sources/leap-formal-mathematics-agentic-kung-2026.md`
