---
title: "Gödel Machines: Self-Referential Universal Problem Solvers"
type: source
created: 2026-05-21
updated: 2026-05-21
sources:
  - godel-machine-schmidhuber-2003.md
arxiv_id: "cs/0309048"
authors:
  - Jürgen Schmidhuber
first_author: Jürgen Schmidhuber
year: 2003
introduces:
  - "[[godel-machine]]"
  - "[[self-improving-agent]]"
tags:
  - 2003
status: complete
importance: high
---

# Gödel Machines: Self-Referential Universal Problem Solvers Making Provably Optimal Self-Improvements

**Source:** `sources/godel-machine-schmidhuber-2003.md`
**arXiv:** [arXiv:cs/0309048](https://arxiv.org/abs/cs/0309048) (v1 Sep 2003; v5 Dec 2006)
**Tech report:** TR IDSIA-19-03
**Author:** [[jurgen-schmidhuber]] (single author)
**Affiliations:** [[idsia]] (Lugano), [[tu-munich]]
**Year:** 2003 (revised through 2006)

## Summary

The Gödel machine is the theoretical foundation for the "agent that can improve itself" research thread. Schmidhuber defines the first mathematically rigorous, fully self-referential, optimally efficient class of general problem solvers: a machine loaded with an initial program `p(1)` that contains (a) a sub-optimal policy `e(1)` for interacting with the environment and (b) a proof searcher that systematically enumerates `(switchprog, proof)` pairs. The machine rewrites any part of its own code — including the proof searcher itself — *as soon as it has found a proof* that the rewrite is useful with respect to a problem-dependent utility function `u` (e.g., expected cumulative future reward). The whole apparatus — hardware semantics, the initial code, environmental priors, costs, and `u` — is encoded as axioms inside `p(1)`, so the searcher can reason about the consequences of modifying any part of itself.

The central result is the **Global Optimality Theorem (Thm 4.1)**: each accepted self-rewrite is *globally* optimal, not greedy, because the target theorem the machine must prove explicitly states that the immediate rewrite has higher utility than *continuing the proof search for any alternative rewrite*. There are no local maxima by construction: the cost of *not* switching implicitly evaluates every alternative `switchprog` the unmodified `p` might ever find later. Inspired by Kurt Gödel's 1931 self-referential formulas, the machine inherits Gödel's incompleteness limits — self-improvements whose utility is unprovable (in the given axiom system, within available resources) are simply ignored, but if no proof exists then "humans cannot do much either."

This paper is the direct theoretical ancestor of the [[darwin-godel-machine]] (Zhang et al. 2025), which makes the same vision empirical 22 years later by replacing the formal proof requirement with *Darwinian* selection (trial-and-keep on a benchmark). Schmidhuber explicitly contrasts the Gödel machine with Hutter's prior universal RL machines (AIXI, AIXI(t,l), HSEARCH) and with Levin's Universal Search and his own OOPS: all of those have *hardwired, unmodifiable* meta-algorithms and therefore suffer permanent O()-hidden constant slowdowns (typically `>> 10^1000`); the Gödel machine, by virtue of true self-reference, can in principle prove and exploit drastic reductions of those constants.

## Key Points

- **Formal self-reference.** The initial code `p(1)` encodes axioms describing the hardware's state transition function, the code of `p(1)` itself (no circularity — software can include a program that prints the software), stochastic environment priors, reward/cost axioms, and the utility function `u`. Proof techniques can therefore reason about the effects of any program — including the running proof searcher.
- **Proof-checker-driven self-rewrite.** Six proof-modifying instructions (`get-axiom`, `apply-rule`, `delete-theorem`, `set-switchprog`, `check`, `state2theorem`) make it impossible to insert an incorrect theorem into the proof — proof verification is trivialized. `check()` is the only path that can flip `switchbit` and transfer control to `switchprog`, which may then rewrite any non-hardware-reserved part of state including `p` itself.
- **Globally optimal by construction — no local maxima.** A self-rewrite is executed only after the machine proves a *target theorem* of the form `u[s ⊕ switchbit=1] > u[s ⊕ switchbit=0]`. The right-hand side already encodes the utility of continuing to search for alternatives, so the accepted switch dominates *all* future alternative rewrites the unmodified searcher would have found.
- **Problem-dependent utility function.** `u` is plug-in: expected cumulative reward (eq. 1), worst-case reward, reward-per-time-interval, fast theorem-proving (reward = 1/proof-time), TSP within time budget, exploration-vs-exploitation tradeoff, etc. Unlike AIXI(t,l), the Gödel machine does not require an enumerable environmental distribution.
- **Asymptotically optimal initial proof searcher (BIOPS), but more than O()-optimal afterward.** Schmidhuber's Bias-Optimal Proof Search (Method 5.1, an online extension of Universal Search) gives the *initial* `p(1)` an optimal order of complexity (Thm 5.1). Crucially, the machine can then prove and execute self-rewrites that eliminate O()-hidden constants — something Hutter's HSEARCH and AIXI(t,l) cannot do because their proof searchers are hardwired.
- **Limits are exactly Gödel's limits.** Self-improvements whose effectiveness is unprovable within the given axiomatic system (or within the resource budget) must be ignored. Realistic limited-resource Gödel machines also cannot exploit speed-ups whose utility cannot be proved in available time/space.
- **Possible self-improvements catalogued (Sec 6.1).** Re-balancing time between proof search and the environment-interaction subpolicy; modifying only `e`; modifying the axioms themselves; modifying `u` (only if provably better under the *old* `u`); modifying only the order of generated proofs; deleting the proof searcher entirely in trivial environments; even augmenting its own hardware.
- **The foundational paper for the self-improving-agent research thread.** Twenty-two years before LLM-powered agents made empirical self-improvement tractable, this paper established the formal vocabulary — self-reference, target theorem, proof-driven self-rewrite, global optimality of accepted modifications — that the [[darwin-godel-machine]] and related [[self-improving-agent]] systems later instantiate empirically.

## Entities

- [[jurgen-schmidhuber]] — sole author; long-standing program on metalearning, self-modifying policies, and universal search culminating in this paper
- [[idsia]] — Schmidhuber's lab in Lugano; primary affiliation on the tech report
- [[tu-munich]] — Schmidhuber's secondary affiliation (TU München)

## Concepts

- [[godel-machine]] — the canonical concept page; this paper is its introduction
- [[self-improving-agent]] — the broader research thread the Gödel machine founds
- [[darwin-godel-machine]] — the empirical 2025 descendant that replaces formal proofs with Darwinian selection on a coding benchmark
- [[universal-search]] — Levin's hardwired predecessor; BIOPS is an online extension of it applied to proof space
- [[reinforcement-learning]] — the Gödel machine is presented as a provably optimal general RL machine; AIXI and AIXI(t,l) are the contrasted prior universal RL approaches

## Notable Quotes

> "Inspired by Kurt Gödel's celebrated self-referential formulas (1931), such a problem solver rewrites any part of its own code as soon as it has found a proof that the rewrite is useful, where the problem-dependent utility function and the hardware and the entire initial code are described by axioms encoded in an initial proof searcher which is also part of the initial code." *(Abstract)*

> "We show that such a self-rewrite is globally optimal — no local maxima! — since the code first had to prove that it is not useful to continue the proof search for alternative self-rewrites." *(Abstract)*

> "Unlike Hutter's previous non-self-referential methods based on hardwired proof searchers, ours not only boasts an optimal order of complexity but can optimally reduce any slowdowns hidden by the O()-notation, provided the utility of such speed-ups is provable at all." *(Abstract)*

> "These restrictions [Gödel's] should not worry us; if there is no proof of some self-rewrite's utility, then humans cannot do much either." *(Sec 1)*

## References

- Cross-link: [[darwin-godel-machine-zhang-2025]] — the empirical 2025 descendant; explicitly positions itself as a relaxation of this paper's proof requirement
- Contrasted prior work: Hutter's AIXI [16, 19], AIXI(t,l) [16], and HSEARCH [17] — universal RL / fastest-algorithm constructions with *hardwired*, non-self-referential proof searchers; the Gödel machine's central novelty is making the proof searcher itself modifiable
- Predecessor: Levin's Universal Search [24, 26] — asymptotically optimal but unmodifiable meta-algorithm; BIOPS extends it online into proof space
- Companion: Schmidhuber's Optimal Ordered Problem Solver OOPS [48] — bias-optimal program search reused as a submodule for solving sequences of proof-search tasks (Sec 5.2)
- Schmidhuber's own prior metalearning line: success-story algorithm / self-modifying policies [37, 53, 54, 55] — empirical but not provably optimal precursors
