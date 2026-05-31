---
title: Library Learning
type: concept
created: 2026-05-29
updated: 2026-05-29
sources:
  - lilo-grand-2023.md
status: draft
importance: medium
tags:
  - 2023
---

# Library Learning

**Library learning** is the program-synthesis lineage in which an agent grows a *library of reusable code abstractions* by repeatedly solving tasks and then compressing or refactoring its solutions into shared, named primitives. Each newly factored abstraction makes future programs shorter and search cheaper, so the library and the solver bootstrap each other. It is the rigorous, formally grounded counterpart to operational [[skill-library|skill libraries]]: where a skill library accretes whatever code happened to work, library learning explicitly *refactors* the corpus to extract minimal, composable abstractions.

- **DreamCoder** ([[dreamcoder]], Ellis et al. 2020) is the pre-LLM ancestor: wake-sleep Bayesian learning that grows a hierarchy of lambda-abstractions.
- **LILO** ([[lilo-grand-2023|Grand et al. 2023]]) pairs an LLM synthesizer with the Stitch compression algorithm and an auto-documentation step, yielding *interpretable* libraries with human-readable names and docstrings.
- **ReGAL** continues the lineage, refactoring generated programs to discover reusable abstractions without explicit search.
- The compression-driven refactoring step is what distinguishes library learning from mere caching of tools.

## Related

- [[skill-library]] — the operational, non-refactoring analogue.
- [[agent-skills]] — packaging convention these abstractions can feed.

## References

- [[lilo-grand-2023]]
