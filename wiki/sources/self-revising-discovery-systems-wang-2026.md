---
title: "Self-Revising Discovery Systems for Science: A Categorical Framework for Agentic AI"
type: source
created: 2026-06-08
updated: 2026-06-08
sources:
  - self-revising-discovery-systems-wang-2026.md
arxiv_id: "2606.01444"
authors:
  - Fiona Y. Wang
  - Markus J. Buehler
first_author: Fiona Y. Wang
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Self-Revising Discovery Systems for Science: A Categorical Framework for Agentic AI

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-05-31–06-07).

## Summary

The paper (from [[mit]]'s Buehler group) argues that scientific discovery is not just answer generation but *revision of the representational regime* in which evidence, artifacts, operations, and verifiers are typed. It develops a category-theoretic account of agentic discovery: within a fixed regime `b`, system state is a copresheaf `It : Sb → Set` over a schema category, and fixed-regime operations are state updates (endofunctorial only when provenance-preserving). Genuine **discovery** is instead a verified *regime transition* `u : Sb → Sb′`, where old artifacts are transported (via a left Kan extension) and compared with the post-transition state to surface residual content beyond functorial transport. This cleanly separates retrieval, search, and discovery without appealing to subjective "novelty."

A key engineering ingredient is a **description-length gate**: a candidate revision is accepted only when it reduces total description length (an MDL/AIC criterion), so the system commits to a new regime only when the change pays for itself. The framework is instantiated in two systems and presented as both a mathematical language and an engineering specification for self-revising AI discovery.

## Key points

- Reframes discovery as **regime transition**, not in-regime update; uses copresheaves over a schema category and Kan-extension transport to compare pre/post states.
- **Description-length (MDL) gate** accepts only revisions that reduce total DL, giving an objective acceptance criterion in place of subjective novelty.
- **Builder/Breaker** instantiation: a protein-mechanics world model is revised under an MDL gate; the accepted law expresses within-chain flexibility as mode-conditioned elastic compliance.
- **CategoryScienceClaw** instantiation: typed skills, artifacts, open needs, workflow mutations, gates, stress tests, and discourse form a proof-carrying knowledge–computation graph; a fiber-network case records candidate models, rejected alternatives, an AIC gate, and an accepted anisotropic-stiffness surrogate.
- Targets the autonomous "AI scientist" setting: the system itself detects when its representational regime should change and revises it under verification.

## Concepts & entities

- [[mit]] — Buehler group (Laboratory for Atomistic and Molecular Mechanics)
- [[self-improving-agent]] — autonomous detection and revision of one's own representational regime
- [[multi-agent]] — Builder/Breaker and CategoryScienceClaw are multi-agent/workflow systems
- [[skill-acquisition]] — typed skills as first-class graph objects in CategoryScienceClaw

## References

- [arXiv:2606.01444](https://arxiv.org/abs/2606.01444) — Wang & Buehler, 2026
- Source text: `sources/self-revising-discovery-systems-wang-2026.md`
