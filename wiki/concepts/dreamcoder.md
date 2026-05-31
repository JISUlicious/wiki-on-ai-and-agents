---
title: DreamCoder
type: concept
created: 2026-05-29
updated: 2026-05-29
sources: []
status: draft
importance: medium
tags:
  - 2020
---

# DreamCoder

**DreamCoder** (Ellis et al. 2020; PLDI 2021) is the pre-LLM ancestor of [[library-learning]]. It is a program-induction system that grows a hierarchy of reusable lambda-abstractions through a **wake-sleep** Bayesian learning loop, alternating between solving tasks and refactoring its library so that the library and a neural search policy bootstrap each other.

- **Wake phase** — search for programs that solve the current tasks, guided by a learned recognition (neural) model.
- **Abstraction sleep** — compress the discovered programs by extracting common fragments into new named library primitives, growing a hierarchy of abstractions.
- **Dream sleep** — train the recognition model on replayed and fantasized programs so future search is faster.
- Demonstrated learning of interpretable domain libraries (list processing, physics laws, drawing) from scratch, anticipating the LLM-era [[lilo-grand-2023|LILO]] line of work that swaps enumerative search for an LLM synthesizer.

## Related

- [[library-learning]] — the research lineage DreamCoder founds.
- [[lilo-grand-2023]] — LLM-era successor that builds on DreamCoder + Stitch compression.

## References

- Ellis et al., "DreamCoder: Bootstrapping Inductive Program Synthesis with Wake-Sleep Library Learning," [arXiv:2006.08381](https://arxiv.org/abs/2006.08381) (PLDI 2021).
