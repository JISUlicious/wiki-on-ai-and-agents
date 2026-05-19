---
title: Darwin Gödel Machine
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - darwin-godel-machine-zhang-2025.md
status: complete
importance: high
tags:
  - 2025
---

The **Darwin Gödel Machine (DGM)** is an empirical Darwinian variant of Schmidhuber's [[godel-machine]]: an LLM-powered coding agent that iteratively rewrites its own Python scaffold and validates each rewrite against a coding benchmark, accumulating viable agents in a population archive rather than hill-climbing on a single current-best. Introduced by Zhang, Hu, Lu, Lange and [[jeff-clune]] (UBC / Vector / [[sakana-ai]], 2025), the DGM relinquishes the original Gödel machine's requirement that every self-modification be formally proven net-beneficial, and substitutes a "produce, trial, select" loop modeled on natural selection. Over 80 iterations, the system lifts a minimalist Claude-3.5-Sonnet coding agent from **20.0% → 50.0% on SWE-bench Verified** and from **14.2% → 30.7% on Polyglot**, with gains transferring across both models (Claude 3.7: 19% → 59% on SWE-bench) and benchmarks.

## Core mechanism

- **The DGM loop**: select parent from archive → parent reads its own evaluation logs → parent proposes a feature improvement → parent edits its own codebase → child evaluated on SWE-bench / Polyglot → child admitted to archive iff it compiles and retains edit capability.
- **Population archive (open-ended search)**: every viable agent ever generated is retained; parent-selection probability ≈ score × 1/(1 + num_children_with_edit). Every node keeps non-zero selection probability, so deceptive stepping-stone paths remain reachable — iterations 4 and 56 produced agents worse than their parent that ultimately led to the best discovered agent.
- **Frozen FM, evolving scaffold**: foundation-model weights are not updated; what evolves is the surrounding Python codebase (tool definitions, edit primitives, context-management, peer-review loops). Discovered improvements include finer-grained file viewing, string-replacement edits instead of whole-file rewrites, patch-validation retry loops, history-aware patch generation, and multi-sample-with-critic voting.
- **Self-improvement as a coding task**: because the agent *is* a code repository, improving on a coding benchmark is direct empirical evidence of improved self-improvement capability — no separate meta-utility needed.
- **Beats both ablations**: DGM-without-self-improvement (= [[adas]]-style fixed meta-agent) plateaus, and DGM-without-archive (pure hill-climbing) plateaus from bad-mutation contamination. Both self-modification and archived open-ended search are jointly load-bearing.

## What it advances

The DGM is the first paper to close the self-referential loop empirically: the agent that solves the downstream task and the agent that improves the downstream-task-solving agent are the same artifact. The SWE-bench jump from 20% to 50% on Claude 3.5 — discovered by the system itself, without a human meta-designer — is the load-bearing evidence that recursive self-improvement of LLM agent scaffolds is more than a thought experiment. It also explicitly flags the safety problem: open-ended evolution optimized for a benchmark can amplify misalignment, growing uninterpretability, and gaps between benchmark score and safety properties.

## Relation to other concepts

Canonical 2025 instance of [[self-improving-agent]]. Inherits the skill-library / open-ended-discovery lineage from [[voyager]] (skills as archive analogue) and the verbal self-correction lineage from [[reflexion]]. Conceptual descendant of Schmidhuber's [[godel-machine]] with the proof requirement replaced by Darwinian selection. Algorithmic kin to [[quality-diversity]] and [[meta-learning]]; ablation baseline is [[adas]]. Compare to [[agentic-rl]] approaches that update weights rather than scaffold.

## References

- [[darwin-godel-machine-zhang-2025]] — primary source ([arXiv:2505.22954](https://arxiv.org/abs/2505.22954), ICLR 2026).
- Code: <https://github.com/jennyzzt/dgm>
- Theoretical antecedent: Schmidhuber 2007 (Gödel machine).
- Precursors: [[voyager]] (open-ended skill discovery), [[reflexion]] (verbal self-correction).
