---
title: Skill Optimization
type: concept
created: 2026-06-02
updated: 2026-06-02
sources:
  - skillopt-yang-2026.md
  - skillmoo-gong-2026.md
  - skillreducer-gao-2026.md
  - skill1-shi-2026.md
  - sage-skill-library-wang-2025.md
status: complete
importance: high
tags:
  - 2026
---

# Skill Optimization

**Skill optimization** is the 2026 research thread that treats an agent's [[agent-skills|skill]] — typically a [[skill-md-format|SKILL.md]] document plus its bundled resources — as an **artifact to be automatically improved**, rather than something a human authors once and freezes. Where [[skill-acquisition]] asks *how does the agent get a skill at all*, skill optimization asks *given a skill, how do we make it measurably better* — higher task success, lower token cost, better routing — usually in a closed loop driven by held-out evaluation.

The thread crystallized as a recognizable sub-area in the first half of 2026, with a cluster of `Skill*`-named papers. It is the skill-level analogue of two adjacent self-improvement threads already in the wiki: [[agentic-context-engineering]] (evolving the *prompt* as a delta-updated playbook) and [[agentic-harness-engineering]] (evolving the whole *harness*). Skill optimization sits between them — narrower than a harness, more structured than a free-form prompt.

## Two families

### A. Text-space optimization (no weight updates)

Treat the skill document as text and let an LLM propose **bounded edits**, accepting an edit only when a held-out score improves. No gradients, no fine-tuning — the optimization happens entirely in token space.

- [[skillopt-yang-2026|SkillOpt]] (Yang et al., Microsoft Research, 2026) — the centerpiece. A controllable text-space optimizer: scored rollouts → bounded add/delete/replace edits on a single skill document → accept iff held-out validation improves. Borrows gradient-descent machinery as metaphor: a textual "learning-rate" budget, a rejected-edit buffer, epoch-wise meta updates. Ships a compact `best_skill.md` (300–2000 tokens) that adds **zero inference-time calls**. Best-or-tied in 52/52 settings.
- [[skillmoo-gong-2026|SkillMOO]] (Gong et al., 2026) — the **multi-objective** variant: LLM-proposed edits + NSGA-II Pareto selection trading pass-rate against inference cost. Up to 31.7% cost reduction with +21pp pass-rate on SWE tasks.
- [[skillreducer-gao-2026|SkillReducer]] (Gao et al., 2026) — the **efficiency** variant: a two-stage optimizer compressing routing descriptions (~48%) and skill bodies (~39%) while *improving* functional quality. Audits real skills: 26.4% lack routing descriptions; >60% of body content is non-actionable.

### B. RL-based optimization (weight updates)

Train the agent's policy to select / use / accumulate skills better, with task outcome as the reward.

- [[skill1-shi-2026|Skill1]] (Shi et al., 2026) — a single RL policy that co-evolves skill **selection + utilization + distillation** toward a shared task-outcome objective (ALFWorld, WebShop).
- [[sage-skill-library-wang-2025|SAGE]] (Wang et al., Amazon, 2025) — RL that accumulates reusable skills across **sequential** tasks with skill-integrated rewards; lifelong skill-library learning (AppWorld: +8.9% goal completion, −26% steps, −59% tokens).

These connect to the RL-trained memory-operation policies ([[memory-r1]], [[mem-alpha]]) — same "train the policy that manages a text artifact with outcome reward" pattern, applied to skills instead of memory.

## What is being optimized

| Target | Methods | Objective |
|---|---|---|
| The skill **body** (instructions) | SkillOpt, SkillReducer, SkillMOO | task success, token cost |
| The skill **routing description** (L1 metadata) | SkillReducer | retrieval accuracy, token cost |
| Skill **selection** (which skill to use) | Skill1 | task outcome |
| Skill **utilization** (how to use it) | Skill1 | task outcome |
| The skill **library** (set accumulated over time) | SAGE | lifelong task outcome |
| Multi-objective (success × cost) | SkillMOO | Pareto front |

## Why it matters

- **Skills are written by humans and are usually bad.** SkillReducer's audit (26.4% missing routing descriptions, >60% non-actionable body) shows the hand-authored baseline has large headroom. Optimization closes it automatically.
- **Held-out gating prevents overfitting to the prompt.** The defining methodological move (SkillOpt, SkillMOO) is accepting an edit only on validation improvement — borrowed straight from ML training discipline.
- **Zero inference-time cost.** Text-space optimization produces a static improved artifact; unlike RL it needs no special serving path, and unlike multi-agent debate it adds no extra calls at run time.
- **Evaluation is the bottleneck.** [[agentic-skills-in-the-wild-liu-2026|Liu et al. 2026]] shows skill gains measured on hand-picked skills degrade badly when retrieving from ~34,000 real skills — so optimization must be evaluated in realistic skill-pool settings, and query-specific refinement matters.

## Relation to other concepts

- [[agent-skills]] — the artifact being optimized.
- [[skill-acquisition]] — the complementary question (getting vs improving a skill).
- [[skill-md-format]] — the concrete document format the text-space optimizers edit.
- [[agentic-context-engineering]] — prompt-playbook evolution; the closest sibling (both evolve text artifacts via LLM edits + counters). Distinction: ACE evolves the *running context/prompt*; SkillOpt optimizes a *reusable, portable skill document* offline.
- [[agentic-harness-engineering]] — evolves the whole harness; skill optimization is one component of that broader loop.
- [[memory-r1]] / [[mem-alpha]] — the memory analogue of RL-based skill optimization.
- [[skill-evaluation]] — how optimized skills are measured in realistic settings.

## References

- [[skillopt-yang-2026]] — SkillOpt: the controllable text-space optimizer (centerpiece)
- [[skillmoo-gong-2026]] — SkillMOO: multi-objective (success × cost)
- [[skillreducer-gao-2026]] — SkillReducer: token-efficiency compression
- [[skill1-shi-2026]] — Skill1: unified RL over selection/utilization/distillation
- [[sage-skill-library-wang-2025]] — SAGE: RL skill-library accumulation
- [[agentic-skills-in-the-wild-liu-2026]] — realistic skill-usage benchmark (the evaluation caveat)
