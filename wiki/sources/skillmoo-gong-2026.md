---
title: "SkillMOO: Multi-Objective Optimization of Agent Skills for Software Engineering"
type: source
created: 2026-06-02
updated: 2026-06-02
sources:
  - skillmoo-gong-2026.md
arxiv_id: "2604.09297"
authors:
  - Jingzhi Gong
  - Ruizhen Gu
  - Zhiwei Fei
  - Yazhuo Cao
  - Lukas Twist
  - Alina Geiger
  - Shuo Han
  - Dominik Sobania
  - Federica Sarro
  - Jie M. Zhang
first_author: Jingzhi Gong
year: 2026
introduces:
  - "[[skill-optimization]]"
tags:
  - 2026
status: complete
importance: high
---

# SkillMOO: Multi-Objective Optimization of Agent Skills for Software Engineering

**Source file**: [[skillmoo-gong-2026]]
**arXiv**: [arXiv:2604.09297](https://arxiv.org/abs/2604.09297) (v2, 17 May 2026, cs.SE)
**Authors**: Jingzhi Gong (jingzhi.gong@kcl.ac.uk), Ruizhen Gu, Zhiwei Fei, Yazhuo Cao, Lukas Twist, Alina Geiger, Shuo Han, Dominik Sobania, Federica Sarro, Jie M. Zhang
**Affiliations**: [[kings-college-london|King's College London]] (Gong, Cao, Twist, Zhang); [[queens-university-belfast|Queen's University Belfast]] (Gu); [[nanjing-university|Nanjing University]] (Fei); [[johannes-gutenberg-university-mainz|Johannes Gutenberg University Mainz]] (Geiger); [[university-college-london|University College London]] (Han, Sarro); [[university-of-duisburg-essen|University of Duisburg-Essen]] (Sobania)
**Year**: 2026

## Summary

SkillMOO reframes the configuration of [[agent-skills|agent skills]] for software engineering (SE) as a **multi-objective search problem**. Current practice treats skill bundles as static, hand-crafted assets — or, in the recent wave of skill-evolution work, optimizes them on **pass rate alone**. Gong et al. argue this is insufficient: a skill can raise task success while substantially inflating token cost or injecting misleading guidance. SkillMOO instead treats a task-specific **skill bundle** (a subset or edited version of the skills exposed to a coding agent) as a candidate solution and evolves it through a two-agent solver–optimizer loop. A *task solver* agent runs the SE task and records pass rate, cost, runtime, and failure traces; a *skill optimizer* agent reads that evidence and proposes edits — **pruning, substitution, reordering, adding, or rewriting** skill content — to improve the pass/cost tradeoff. Candidate bundles are then ranked by **NSGA-II** non-dominated sorting with crowding-distance tie-breaking on two explicit objectives, `min [−pass(b), cost(b)]`, and the survivors breed the next generation.

This positions SkillMOO as the **cost-aware, multi-objective counterpart to [[skillopt-yang-2026|SkillOpt]]'s single-objective text-edit optimization**: where SkillOpt (and contemporaries like EvoSkill, EvoSkills, and Meta Context Engineering) iteratively rewrites skills to push a single pass-rate objective, SkillMOO keeps an explicit Pareto front trading verifier pass rate against LLM inference cost, and refuses to hide token/runtime tradeoffs behind a pass-rate-only score. Evaluated across all 16 SE tasks in [[skillsbench|SkillsBench]] with [[glm-5|GLM-5]] driving both agents, SkillMOO achieves the **top pass-rate rank on 11 of 12 non-zero-pass tasks**, with cost reductions of **up to 31.7%** over static bundles and pass-rate gains of **up to +21 percentage points**. An analysis of 38 filtered skill edits finds that pruning and substitution — not adding guidance — dominate the successful operations, motivating a new class of **cost-aware, search-based skill engineering** ([[search-based-software-engineering|SBSE]] for skills).

## Method: The Solver–Optimizer Loop

A candidate is a task-specific skill bundle. Search proceeds in three repeating phases over a small evolutionary population:

1. **Initialization (Generation 0)** — the population is seeded with candidates sampled as **diverse random subsets** of the available skill pool. The task solver executes the SE task for each candidate and records pass rate, cost, runtime, and failure traces. (Because Gen 0 is already random subsets, a separate random-search baseline is omitted — any SkillMOO advantage over Gen 0 reflects the value of the LLM-guided edits + NSGA-II selection added on top.)
2. **Optimization** — the skill optimizer agent receives each surviving bundle plus its evaluation evidence (including error traces) and decides the single edit most likely to improve the pass/cost tradeoff: removing distracting guidance, replacing a misaligned skill, adding missing task-specific instructions, reordering exposed skills, or rewriting stale content. Each child is re-evaluated by the same solver.
3. **Selection** — the generation's evaluated candidates are ranked by **NSGA-II** [Deb et al.] non-dominated sorting with crowding-distance tie-breaking on the bi-objective formulation `min f(b) = [−pass(b), cost(b)]`, where `pass(b)` is verifier pass rate and `cost(b)` is LLM inference cost. Top-ranked survivors breed the next generation.

The framing explicitly distinguishes SkillMOO from skill **discovery** or **routing** (e.g., SkillRouter): it asks not only *which skill helps* but *which bundle offers the best pass/cost tradeoff*.

## Experimental Setup

- **Benchmark**: all 16 SE tasks in [[skillsbench|SkillsBench]] (categories span info retrieval, build repair, code migration, performance optimization, ML reproduction, data engineering, etc.). Bundle sizes range from 1 to 8 skills per task.
- **Verifier hardening**: SkillsBench's default test coverage is minimal, so all tasks were expanded to **≥40 tests** via GPT-5.4, generated independently per task without access to skill content, manually validated, and applied uniformly across all three compared methods; compile/build gates were preserved as model-independent ground truth.
- **Model**: [[glm-5|GLM-5]] for both the task solver and skill optimizer agents (chosen for cost-to-capability ratio and API availability).
- **Budget**: population size 4, 3 generations, 900 s timeout per candidate; per-candidate evaluation costs $0.10–$14 (USD at GLM-5 pricing), keeping per-task optimization overhead practical ($1–$15 and under three hours).
- **Statistics**: each task run 10× per method; **Scott-Knott Effect Size Difference (ESD)** ranks computed on the 10-run pass-rate vectors. RQ2 uses per-task min-max-normalized **hypervolume (HV)** with reference point (0, 1.1).
- **Baselines / RQs**: compared against `ori_skill` (original static bundle) and `no_skill` (no guidance). RQ1 = effectiveness on pass rate and cost; RQ2 = is optimization economically worthwhile after overhead; RQ3 = what edit patterns reveal.

## Key Points

- **Multi-objective framing is the core thesis** — SE agent skill bundles should be treated as multi-objective search objects, with pass rate and deployment (inference) cost as *competing* objectives rather than an afterthought, challenging the prevailing pass-rate-only evaluation of skills.
- **LLM-proposed edits + NSGA-II Pareto selection** — a two-agent solver–optimizer loop where the optimizer proposes semantic skill edits and NSGA-II survivor selection keeps the Pareto-best bundles on `[−pass, cost]`.
- **Top pass-rate rank on 11 of 12 non-zero-pass tasks** (4 of the 16 tasks yield zero pass for all methods, excluded due to heavy/specialized runtime dependencies stressing container memory and build budgets).
- **Up to 31.7% cost reduction and +21 pp pass rate** — the strongest non-trivial gain is `fix-build-agentops`: pass rate rises 0.16 → 0.37 (**+21 pp, +131.2% relative**) while cost falls **31.7%** and runtime falls 23.6%, with Scott-Knott ESD confirming rank separation.
- **`no_skill` is cheapest but weaker** — the no-guidance baseline takes rank-1 cost on every task (no skill-content context overhead) but trails SkillMOO on pass rate for 11 of 12 non-zero-pass tasks; it leads pass rate only on `gh-repo-analytics`. This is the empirical case for *why* skills must be optimized multi-objectively: pass-rate-only reporting hides token and runtime tradeoffs.
- **HV improves on all 12 reported tasks** — optimization cost ranges $1.76–$13.73 per task; each one-percent HV gain costs well under two U.S. cents in search fees, so the overhead is worthwhile **when amortized over reuse** (CI jobs, repeated production calls); single-shot tasks justify search only when reliability gains cover the bill.
- **Pruning and substitution dominate successful edits** — from 38 filtered edits: *Remove Skills* is most frequent (16 edits — 11 "drop peripheral/misaligned" at 8/11 pass↑, 9/11 cost↓; 5 "remove redundant" at 5/5 pass↑); *Replace Skills* 7 edits (5/7 pass↑, 7/7 cost↓). **Add Skills shows 0/9 pass improvements** — adding guidance did not help in the logged cases. *Edit Skill Content* is most targeted (removing deprecated API mentions: 2/2 pass↑).
- **A new SBSE problem** — the paper positions cost-aware skill engineering as a new search-based software engineering subfield; RQ3 evidence is explicitly *exploratory/descriptive*, not causal.
- **Threats to validity** — single benchmark (16 tasks); LLM-generated test expansion shares a model family with the solver (mitigated by preserved compile/build gates); results reported for a single model (GLM-5), so transfer to GPT/Claude coding agents is an open question.
- **Research agenda** — causal evaluation of skill edits via controlled ablations; cost-aware skill routing/reuse of learned Pareto fronts; cross-task and cross-model transfer.

## Entities Mentioned

- [[kings-college-london]] — primary affiliation (Gong, Cao, Twist, Zhang)
- [[university-college-london]] — Han, Sarro
- [[queens-university-belfast]] — Gu
- [[nanjing-university]] — Fei
- [[johannes-gutenberg-university-mainz]] — Geiger
- [[university-of-duisburg-essen]] — Sobania
- [[glm-5]] — the LLM driving both the task solver and skill optimizer agents
- [[skillsbench]] — the SE-task benchmark used for all evaluation (16 tasks)

## Concepts Discussed

- [[skill-optimization]] — the concept this source anchors: iteratively improving agent skill bundles via automated edits; SkillMOO is the *cost-aware, multi-objective* variant
- [[agent-skills]] — the artifacts being optimized (loadable instruction/code/resource bundles)
- [[skill-md-format]] — the SKILL.md packaging that skill edits (rewrite, prune sections) operate on
- [[skill-library]] — the available skill pool from which bundles are sampled and edited
- [[swe-bench]] — the canonical SE-agent benchmark; SkillsBench's SE tasks are the closer evaluation surface here
- [[evolutionary-algorithm]] / **NSGA-II** — the genetic survivor-selection algorithm (non-dominated sorting + crowding distance) driving Pareto selection
- [[multi-objective-optimization]] — the bi-objective `[−pass, cost]` formulation and Pareto/hypervolume analysis
- [[search-based-software-engineering]] — the SBSE/AI4SE framing the paper situates itself within
- [[skillopt-yang-2026|SkillOpt]] — the **single-objective sibling**: text-edit skill optimization on pass rate alone; SkillMOO is its cost-aware, multi-objective counterpart

## Notable Quotes

> "We argue that SE agent skill bundles can be treated as multi-objective search objects and present SkillMOO, a framework that evolves skill bundles through LLM-proposed edits and NSGA-II Pareto selection on pass rate and inference cost."

> "This distinguishes SkillMOO from skill discovery or routing: it asks not only which skill helps, but which bundle offers the best pass/cost tradeoff."

> "These patterns suggest that the observed SkillMOO gains often come from removing irrelevant guidance and replacing misaligned content, rather than adding new instructions."

> "The current practice of deploying skills without cost-aware validation leaves better skill configurations unexplored, motivating a new class of cost-aware, search-based skill engineering."

## References

_Original source: `sources/skillmoo-gong-2026.md`_
_arXiv: [arXiv:2604.09297](https://arxiv.org/abs/2604.09297) (v2, 2026-05-17, cs.SE)_

Key related work cited:
- SkillsBench — first systematic paired evaluation of skills across diverse tasks → [[skillsbench]]
- SWE-Skills-Bench — finds many public SWE skills provide little benefit / add token overhead
- SkillRouter — frames skill usage as a retrieval/routing problem over large skill pools
- EvoSkill, EvoSkills, Meta Context Engineering — pass-rate-only skill-evolution approaches → contrast with [[skill-optimization]]
- ACE, GEPA, Meta-Harness — adjacent context/prompt/harness optimization
- LLM-guided Genetic Improvement — closest prior: LLM semantic edits + search-based software evolution (but not on skill bundles)
- Deb et al. — NSGA-II ([[evolutionary-algorithm]])
- Scott-Knott ESD — the rank-comparison statistical method
