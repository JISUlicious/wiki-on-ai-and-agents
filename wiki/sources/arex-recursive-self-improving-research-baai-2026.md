---
title: "AREX: Towards a Recursively Self-Improving Agent for Deep Research"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - arex-recursive-self-improving-research-baai-2026.md
arxiv_id: "2607.21461"
year: 2026
authors:
  - Shuqi Lu
  - Chaofan Li
  - Kun Luo
  - Zheng Liu
  - AREX Team (BAAI)
tags:
  - 2026
  - paper
status: complete
importance: high
---

# AREX: Towards a Recursively Self-Improving Agent for Deep Research

**AREX Team — Shuqi Lu, Chaofan Li, Kun Luo, Zheng Liu (project lead) et al.** (Beijing Academy of Artificial Intelligence, BAAI) — [arXiv:2607.21461](https://arxiv.org/abs/2607.21461). v2, 24 Jul 2026, cs.AI. Models on HuggingFace under `BAAI/arex`.

## Summary

AREX's premise is a stated **discovery–verification asymmetry**: "Discovering an answer that jointly satisfies all constraints is costly because it requires navigating a large and sparsely informative search space, whereas evaluating a proposed candidate can often be decomposed into substantially simpler constraint-wise checks." The paper's move beyond prior uses of verification is precise — existing work uses verification either to *rank* completed candidate trajectories or to refine decisions *within* one trajectory; AREX makes verification **define the transition between research rounds**. A provisional answer is converted into a *partially verified state*, so the agent preserves established progress, isolates residual uncertainty, and formulates a more targeted next research problem. Verification becomes an active control signal, not a final filter.

Architecturally this is a bi-level loop. The **inner research loop** analyses the current objective, calls search/visit/python, integrates evidence, and emits a structured `finish` result: a provisional answer, supporting evidence, and an answer-level **confidence score**. The **outer self-improvement loop** applies a three-way decision rule on that score: `Accept` if `s ≥ τ`; otherwise it runs a trajectory assessment returning a recoverability bit `v` — `Refine` if `v = 1` (preserve reliable findings P, list open issues I, convert them into the next objective, and reinitialise from a refreshed state) or `Restart` if `v = 0` (the trajectory is too noisy to salvage; discard it and reinitialise from the original problem alone). The process is bounded by a max round count and returns the highest-confidence completed answer if nothing clears τ.

The third component is **Autonomous Context Updating (ACU)**: a learned `update_context` tool the agent invokes on itself, compressing its own interaction history into a compact "improvement state" holding verified findings, current candidates, unresolved constraints, validity concerns, rejected candidates, and a next-step plan. The paper stresses this is *not* generic summarization by an external model — the update is produced by AREX itself and organized around its current research objective, keeping the compressed state aligned with its evolving beliefs. Training runs in stages on verified synthetic tasks and teacher trajectories: progressive multi-round capability mid-training (browse-intensive first, then reasoning-intensive), key-step focused supervision, and step-aware RL. The sparse-reward problem in long trajectories is attacked by increasing training exposure to **key steps** — where decisive evidence is acquired, contradictions resolved, or an incorrect research direction repaired. Two instantiations: **AREX-Turbo** (dense 4B, Qwen3.5-4B backbone) and **AREX-Base** (122B-A10B MoE, Qwen3.5-122B-A10B backbone).

## Key points

- **Main results** (Table 1; BrowseComp/GAIA/xbench-2510/HLE = accuracy, DeepSearchQA = F1, WideSearch-en = Item-F1):

| Model | BrowseComp | GAIA | xbench-2510 | DeepSearchQA | WideSearch-en | HLE (tool) |
|---|---|---|---|---|---|---|
| Gemini-3.1-Pro | 85.9 | 80.6 | 53.0 | **93.3** | 66.4 | 51.4* |
| Opus-4.6 | 83.7 | – | – | 91.3 | 77.5 | 53.0* |
| Kimi-K2.6 | 83.2 | 80.6 | **90.0** | 92.5 | 80.8 | 54.0* |
| DeepSeek-V4-Pro | 83.4 | – | 80.0 | 88.7 | 78.0 | 48.2 |
| MiroThinker-H1 | 88.2 | 88.5 | 72.0 | 80.6 | – | 47.7 |
| Qwen3.5-397B | 78.6 | 83.5 | 61.0 | 82.1 | 74.0 | 48.3 |
| Qwen3.5-122B (backbone) | 63.8 | 81.6 | – | – | 60.5 | 47.5 |
| **AREX-Base (122B-A10B)** | 82.5 | 85.4 | 71.0 | 89.9 | **82.0** | **52.4** |
| **AREX-Turbo (4B)** | 70.7 | 81.6 | 57.0 | 78.5 | 68.5 | 40.6 |

  (* = full HLE; unmarked = text-only subset — so the HLE column is not apples-to-apples across rows.) AREX-Base posts the **best reported WideSearch-en score (82.0)** and the best text-only HLE (52.4), and lifts its own 122B backbone on BrowseComp by **+18.7 points** (63.8 → 82.5). The 4B Turbo beats Qwen3.5-35B on five of six benchmarks.
- **The two loops are worth 22.9 points, decomposed** (BrowseComp, Table 3): no ACU + no outer loop **59.6** → ACU only **71.4** (+11.8) → ACU + outer loop **82.5** (+11.1). Without ACU, the outer loop alone gives 59.6 → 69.8 (+10.2). Both halves contribute roughly equally and near-additively.
- **The confidence score is a usable control signal**, which is what makes the accept/refine/restart rule work: with ACU, **95.9%** of correct outputs land in the 90–100 confidence bin, while **55.2%** of incorrect outputs fall below 60 (89.3% / 61.0% without ACU). Many failures are thus identifiable from the answer-level score without re-reading the trajectory.
- **ACU is used proactively, not as an overflow valve.** On BrowseComp the agent invokes `update_context` in **80.3%** of cases, at a mean active context of **25,721 tokens** (median 25,386) against a **128K** window — only **0.01%** of updates happen at or above the limit. Triggers: revise search strategy **66.9%**, reject candidate **13.6%**, identify new lead 7.2%, summarize progress 6.4%, verify evidence/answer 5.3%. Contents preserved (multi-label): next-step plan **96.4%**, unresolved constraints **95.5%**, rejected candidates **81.5%**, verified findings **72.1%**, current candidates 39.2%, validity concerns 14.1%. Note the priority ordering: *what still needs checking* and *what to do next* are preserved more consistently than the findings themselves.
- **Key steps really are underlearned.** After full-trajectory mid-training, average token-level NLL is 0.232 on ordinary steps vs. 0.277 (evidence discovery), 0.298 (path rejection and redirection) and 0.300 (key context-update) — relative increases of ~19%, ~28% and ~29%. This is the empirical justification for concentrating supervision there.
- **Training-recipe ablations** (BrowseComp, full AREX = 82.5): replacing key-step focused supervision with equal-budget random-step replay → **74.1** (largest drop, −8.4); replacing progressive multi-round capability training with direct mixed training → **77.5** (−5.0); replacing step-aware RL with standard GRPO → **79.4** (−3.1).
- **Budget**: up to **300 inner-loop turns** and **5 outer self-improvement operations** per episode.
- An appendix reports an exploratory **self-distillation** experiment where trajectories from an intermediate agent supervise the same 122B-A10B backbone more effectively than the original teacher trajectories, but the authors explicitly present it as a future direction, not a component of the final system, and note self-generated trajectories may inherit or amplify teacher biases.

## Concepts & entities

- [[verifier]] — the paper is a direct application of the verify-is-cheaper-than-discover asymmetry, but with a twist worth noting: verification here is **constraint-wise decomposition performed by the agent itself**, producing a partially verified state that seeds the next round, rather than an external checker gating outputs.
- [[self-improving-agent]] — AREX's "recursive self-improvement" operates **within a single query episode** (recursive refinement of the answer and research state), not across tasks via weight or code self-modification.
- [[agentic-rl]] — step-aware RL over long-horizon research trajectories, with critical-interval exposure as the credit-assignment fix; standard GRPO is the weaker baseline it replaces.
- [[memory-management]] — ACU is an agent-invoked context compression tool; the finding that it fires at ~20% of the window rather than at overflow makes it a research operation rather than an eviction policy.
- [[agentic-context-engineering]] — the improvement state (verified findings / candidates / unresolved constraints / validity concerns / rejected candidates / next plan) is an explicit context schema the model learns to write.
- [[long-context-llm]] — motivating tension: retaining the full 128K history distracts subsequent reasoning, while indiscriminate truncation discards evidence needed for later verification.
- [[outcome-based-reward]] — the sparse terminal reward is exactly what step-aware supervision is introduced to compensate for.
- [[agent-evaluation]] — evaluated on BrowseComp, GAIA, xbench-DeepSearch-2510, DeepSearchQA, WideSearch, and HLE-with-tools.

## References

- [arXiv:2607.21461](https://arxiv.org/abs/2607.21461)
- Source text: [[arex-recursive-self-improving-research-baai-2026]]
- Homepage: `https://vectorspacelab.github.io/arex-model/` · Models: `https://huggingface.co/collections/BAAI/arex`
