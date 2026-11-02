---
title: "Bounded Loops: Pre-Run Spend Bounds, Proved Termination, and Verified Completion for Agent Harnesses"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - bounded-loops-agent-harnesses-bhardwaj-2026.pdf
arxiv_id: "2609.27871"
authors:
  - Varun Pratap Bhardwaj
  - Garima Singh
  - Arun Pratap Bhardwaj
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Bounded Loops: Pre-Run Spend Bounds, Proved Termination, and Verified Completion for Agent Harnesses

Bhardwaj et al. [arXiv:2609.27871](https://arxiv.org/abs/2609.27871), 2026.

## Summary

A bounded loop is a worker, an independent gate the worker cannot write to, and a declared budget. The paper proves termination, gate-before-done and spend guarantees, and measures gates against mutants.

## Key points

- **Self-attestation.** A check is self-attested if its subject controls the value being checked. Such a check can pass while the property fails.
- **Vacuity.** A gate is vacuous if absent or empty artifacts satisfy it. A 69-loop catalogue contained 47 vacuous gates in shipped, reviewed code.
- **Model judges.** Combined by conjunction with a deterministic gate, a judge can't raise false accepts and can only add false rejects. Persuading the judge therefore doesn't move the deterministic gate.
- **Measurement caution.** No false accepts over 209 mutants after repair, but a fresh mutant family recovered a 23.3% false-accept rate. The authors call the first figure saturation, not quality.

## What it does not show

The catalogue is the authors' own engine, and the paper doesn't compare outcomes across products. The guarantees are about the harness and its gates, not about whether a gate tests the right thing.

## Concepts & entities

- [[false-success]] · [[verifier]] · [[guides-and-sensors]] · [[harness-extension-and-goal-loop-axes]]

## References

- [arXiv:2609.27871](https://arxiv.org/abs/2609.27871)
