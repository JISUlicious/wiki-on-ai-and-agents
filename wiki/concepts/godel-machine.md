---
title: Gödel Machine
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - godel-machine-schmidhuber-2003.md
status: complete
importance: high
tags:
  - 2003
---

# Gödel Machine

A **Gödel machine** is a self-referential problem solver that rewrites its own code when it can *prove* the rewrite is useful, with respect to a problem-dependent utility function. Introduced by Jürgen Schmidhuber in 2003 ([[godel-machine-schmidhuber-2003]]); the direct theoretical ancestor of [[darwin-godel-machine]], which makes the vision empirical 22 years later.

## Core mechanism

- **Self-referential**: the machine has axiomatic descriptions of its hardware, utility function, and initial code — including the proof searcher itself.
- **Proof-driven self-rewrite**: a proof searcher systematically tests computable proof techniques. A self-rewrite executes only after the machine proves it is useful w.r.t. expected future utility.
- **Globally optimal** (no local maxima): the target theorem for any rewrite must dominate *all* future alternative rewrites; the code first proves it is not useful to continue the search.
- **Asymptotically optimal + better**: matches Hutter's BIOPS / HSEARCH on order-of-complexity but can eliminate `O()`-hidden slowdowns when their utility is provable.

## What it advances

- **First mathematically rigorous** framework for self-improving agents — distinguishing *provably good* self-rewrites from heuristic ones.
- Resolves the chicken-and-egg of self-improvement: how can a system know its self-modification is beneficial? The Gödel-machine answer: require a proof before applying.
- Inspires every later self-improving-agent research program; [[darwin-godel-machine]] empirically replaces the proof checker with an evaluation harness + population archive.

## Limitations

- **Proof requirement is impractical in practice.** Modern systems (DGM, Voyager, Reflexion) substitute *empirical validation* (does the rewrite improve a benchmark?) for the Gödel-machine's proof requirement.
- Gödel's incompleteness limits which utility-improving rewrites are provable at all.

## Relation to other concepts

- [[darwin-godel-machine]] — 2025 empirical Darwinian variant.
- [[self-improving-agent]] — the umbrella; Gödel machines are the theoretical anchor.
- [[voyager]] / [[reflexion]] — softer self-improvement precursors that don't require proofs.
- [[universal-search]] / [[reinforcement-learning]] — adjacent universal-optimization frameworks.

## References

- [[godel-machine-schmidhuber-2003]] — Schmidhuber's foundational paper.
- [[darwin-godel-machine-zhang-2025]] — the 2025 empirical descendant.
