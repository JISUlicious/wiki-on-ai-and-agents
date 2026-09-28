---
title: Universal Search
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - godel-machine-schmidhuber-2003.md
status: draft
importance: low
---

# Universal Search

**Universal Search** is Leonid Levin's general-purpose program-search method, the classic example of an *asymptotically optimal* problem solver. In this wiki it matters for one property: its meta-algorithm is **hardwired**. The search procedure cannot inspect or rewrite itself, so the constant-factor slowdowns hidden inside its O() bound are permanent. Overcoming that limitation is the stated motivation for Schmidhuber's [[godel-machine]].

## In this wiki

The page exists because of its place in the prehistory of self-improving agents, as told by [[godel-machine-schmidhuber-2003]]:

- **The hardwired predecessor.** Schmidhuber sets the [[godel-machine]] against Levin's Universal Search, his own OOPS (Optimal Ordered Problem Solver), and Hutter's AIXI(t,l) and HSEARCH. All of them have *unmodifiable* meta-algorithms, so they keep the hidden constant slowdowns inside their O() bounds.
- **BIOPS extends it.** The Gödel machine's initial proof searcher, Bias-Optimal Proof Search, is described as an online extension of Universal Search applied to proof space. It starts with Universal Search's order-of-complexity optimality and can then prove and execute self-rewrites that remove those constants.
- **The lineage continues empirically.** The [[darwin-godel-machine]] replaces the proof requirement with benchmark-based selection. That makes Universal Search two steps removed from today's [[self-improving-agent]] systems.

## Related

- [[godel-machine]] · [[darwin-godel-machine]] · [[self-improving-agent]]
- [[jurgen-schmidhuber]] · [[reinforcement-learning]] · [[meta-learning]]

## References

- [[godel-machine-schmidhuber-2003]]
