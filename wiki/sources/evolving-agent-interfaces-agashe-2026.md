---
title: "Evolving Agent Interfaces (Agent S4)"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - evolving-agent-interfaces-agashe-2026.md
arxiv_id: "2610.28601"
authors:
  - Saaket Agashe
  - Yuzhe Yang
  - Gonzalo Gonzalez-Pumariega
  - Bhuvan Koduru
  - Chih-Lun Li
  - Jiachen Yang
  - Ang Li
  - Xin Eric Wang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Evolving Agent Interfaces (Agent S4)

**Saaket Agashe, Yuzhe Yang, Gonzalo Gonzalez-Pumariega, Bhuvan Koduru, Chih-Lun Li, Jiachen Yang, Ang Li, Xin Eric Wang** (Simular; first authors from [[uc-santa-barbara|UCSB]], with Cornell and [[cmu|CMU]] contributors; work done at Simular) — [arXiv:2610.28601](https://arxiv.org/abs/2610.28601), v1, cs.AI, 20 Oct 2026. Code: `simular-ai/agent-s`.

## Summary

Agent S4 argues that the **interface** of a [[computer-use-agent]] should itself be evolved, not hand-designed. The interface covers the observation and action primitives plus the knowledge of when to use each. A **meta-agent** reads a task agent's trajectories on synthetic training tasks, with no success labels, and writes or revises a software repository of interfaces and usage knowledge. The interfaces cover system-wide, app-specific, cross-app and task-specific operations. **Population-based evolution** (mutation and crossover, binary tournaments) then selects repositories by their score on 15 official validation tasks that the meta-agent never sees. The task agent and the meta-agent are the *same* Qwen3.8-27B in a modified [[prime-agent-self-improving-rlm-harness-karten-2026|Prime Agent]] harness with a persistent IPython kernel. The training tasks (30 for OSWorld V2, 48 for ScienceBoard) were authored by GPT-6 Astra and GPT-5.6 Sol.

## Key points

- **OSWorld V2 (108 tasks), binary success.** The evolved repository (no scratch VM) scores **21.30%** at $1.10 per task. The Qwen CUA baseline (same model, fixed interfaces) scores 5.56%, the "3.8×". Reported results for Claude Opus 4.8 batched are 20.60% at $72.40, and GPT-5.5 batched 13.00% at $25.50.
- **The bridge from 5.56 to 21.30 has three steps** (Table 4):
  - Prime Agent harness with **no repository**: 15.74%.
  - Plus the hand-written ~550-line **seed repository**: 19.44%.
  - Plus **evolution**: 21.30%.
- **Partial score.** Evolved repositories reach 46.90% (no VM) or 51.06% (with scratch VM), both below Opus 4.8's 54.80. The scratch VM improves partial progress but not binary success.
- **ScienceBoard (169 tasks):** Qwen CUA 41.4 → seed repository 55.03 → evolved **58.58** (99 vs 93 tasks solved). Model calls per task fall from 61.7 to 58.9.
- Taxonomy of evolved interfaces: system-wide, app-specific (LibreOffice ranges, REAPER tracks, ChimeraX molecules), cross-app (PDF reading, web pages), and task-specific procedures with built-in completion checks.

> [!warning] Newsletter vs paper
> The numbers match the abstract, but both headline gaps **mostly measure harness and seed, not evolution**.
> - **OSWorld V2.** "3.8× the same model with fixed interfaces" compares against the Qwen CUA scaffold. The same model in the authors' harness with *no* repository already gets 15.74%. Evolution itself adds **+1.86 pp (about 2 of 108 tasks)** over the human-written seed.
> - **ScienceBoard.** 41.4 → 58.6 is likewise mostly seed (+13.6); evolution adds +3.55.
> - **Frontier comparisons.** The GPT-5.5 and Opus 4.8 numbers are quoted from the OSWorld V2 paper, not re-run.

## What it does not show

- **Selection on test.** Full-benchmark results "include the 15 validation tasks of each" benchmark, and those are the tasks used to select the repository.
- **One run per strategy, no seeds.** No denoising on OSWorld V2, where a 2-task difference decides the headline.
- **No ablation of the meta-agent's model.** Evolution with a stronger meta-agent is untested, as is transfer of an evolved repository to a different task model.

## Concepts & entities

- [[computer-use-agent]] · [[gui-agent]] · [[agent-computer-interface]] · [[osworld]] — interface design as an agent capability.
- [[self-improving-agent]] · [[quality-diversity]] · [[agentic-harness-engineering]] — population-based evolution of an artifact, not of weights.
- [[tool-creation]] · [[skill-library]] · [[codeact]] — evolved interfaces are code tools plus usage notes.
- [[prime-agent-self-improving-rlm-harness-karten-2026]] (harness) · [[claude-opus-4-8]] · [[qwen]] · [[uc-santa-barbara]].
- Related: [[anyact-universal-action-self-evolving-agents-xu-2026]] (also adds MCP/API actions to OSWorld-derived tasks), [[harness-vs-model-attribution]] (why the seed vs evolution split matters).

## References

- [arXiv:2610.28601](https://arxiv.org/abs/2610.28601)
- Code: https://github.com/simular-ai/agent-s
