---
title: "SkillOpt: Executive Strategy for Self-Evolving Agent Skills"
type: source
created: 2026-06-02
updated: 2026-06-02
sources:
  - skillopt-yang-2026.md
arxiv_id: "2605.23904"
authors:
  - Yifan Yang
  - Ziyang Gong
  - Weiquan Huang
  - Qihao Yang
  - Xue Yang
  - Chong Luo
first_author: Yifan Yang
year: 2026
introduces:
  - "[[skill-optimization]]"
tags:
  - 2026
status: complete
importance: high
---

# SkillOpt: Executive Strategy for Self-Evolving Agent Skills

**Source**: `sources/skillopt-yang-2026.md`
**Authors**: Yifan Yang, Ziyang Gong, Weiquan Huang, Qihao Yang, Ziwei Zhou, Zisu Huang, Yan Li, Xuemei Gao, Qi Dai, Bei Liu, Kai Qiu, Yuqing Yang, Dongdong Chen, Xue Yang, Chong Luo. Equal-contribution first authors: Yifan Yang, Ziyang Gong, Weiquan Huang, Qihao Yang, Ziwei Zhou, Zisu Huang. Corresponding authors: **Yifan Yang** ([[microsoft|Microsoft]]) and **Xue Yang** ([[shanghai-jiao-tong-university|SJTU]]).
**Affiliations**: Microsoft (1), [[shanghai-jiao-tong-university|Shanghai Jiao Tong University]] (2), Tongji University (3), Fudan University (4).
**arXiv**: [arXiv:2605.23904](https://arxiv.org/abs/2605.23904) (v2, 2026-05-25) · cs.AI · Code: `https://aka.ms/SkillOpt`

## Summary

**SkillOpt** is, by its authors' claim, the first systematic **controllable text-space optimizer for agent skills**. Its thesis is that a [[skill-md-format|SKILL.md]]-style natural-language skill document should be *trained* with the same discipline that makes weight-space optimization reproducible, rather than hand-crafted, generated one-shot, or evolved through loosely controlled self-revision. SkillOpt treats the skill document as the **external state** of a frozen target model and improves it through bounded, LLM-proposed `add/delete/replace` edits that are accepted **only when they strictly improve a held-out validation score**. A *separate* frontier optimizer model turns scored rollout trajectories into structured edits; the target model, harness, and evaluator stay fixed. The deployed output is a compact `best_skill.md` of roughly 300–2,000 tokens that adds **zero inference-time model calls** — it is just text prepended to the agent context.

The paper makes the deep-learning analogy operational, not decorative. The skill document plays the role of the parameter vector; trajectory-derived edit directions play the role of the gradient; a **textual "learning rate"** — the edit budget `Lt`, the maximum number of skill edits per step, with constant/linear/cosine/autonomous schedules — bounds how far one skill version may move from the previous one; the **held-out selection gate** plays the role of validation; and an **epoch-wise slow/meta update** acts like a momentum term, carrying stable editing directions across epochs into a protected region of the skill that step-level edits cannot overwrite. Rejected edits are not discarded: they feed a **rejected-edit buffer** that supplies negative feedback to later optimizer calls within an epoch, so the loop learns from what failed without adding deployment cost. The optimizer-side **meta skill** (a teacher-only summary of which edit patterns helped or failed) is never shipped with the deployed artifact, keeping it compact and portable.

Empirically the results are near-universal. Across **six benchmarks, seven target models, and three execution harnesses** (direct chat, [[claude-code|Codex]]-style, and [[claude-code|Claude Code]]-style loops), SkillOpt is **best or tied-best on all 52 evaluated (model, benchmark, harness) cells**, beating every per-cell competitor among human-written, one-shot LLM, Trace2Skill, [[agentic-context-engineering|TextGrad/GEPA]] prompt-optimizers, and the harness-side EvoSkill. On **GPT-5.5** it lifts the six-benchmark average from 58.8 (no skill) to 82.3 — **+23.5 points** in direct chat — and by **+24.8** inside the Codex loop and **+19.1** inside Claude Code, also clearing the strongest per-cell baseline by +5.4 points (an oracle that picks the best of six methods per cell). The learned artifacts transfer across model scales, across the Codex↔Claude Code harness boundary (a Codex-trained spreadsheet skill gains +59.7 on Claude Code), and to a nearby math benchmark (OlympiadBench→Omni-MATH) without further optimization — all without touching model weights.

## Key Points

- **The optimizer loop** (Sections 3.2–3.6): forward pass = run a **rollout batch** from the train split with the current skill; backward pass = **minibatch reflection** where the optimizer separates failures from successes and proposes structured `add/delete/replace` edits; **bounded text update** = rank/clip the merged edit pool to the top `Lt` edits; **validation gate** = accept the candidate skill iff its held-out selection score is *strictly* greater than the current (ties are rejected, so the deployed skill never silently drifts).
- **Textual learning rate (`Lt`)** is the central control: the max number of skill edits per step, default `Lt = 4` with cosine decay (floor 2). Bounded updates preserve continuity and beat uncontrolled rewriting; ablating the budget (the "without lr" / full-rewrite setting) drops scores (84.6/75.7/57.3 vs gated runs).
- **Rejected-edit buffer**: failed edits and the score drops they caused are retained as an epoch-local negative-feedback record fed to later optimizer calls. Removing it lowers scores by 1.6 / 4.6 / 2.4 points on SearchQA / SpreadsheetBench / LiveMath.
- **Epoch-wise slow/meta update**: longitudinal guidance written into a *protected* skill region; the teacher-only **meta skill** summarizes accepted/rejected patterns but is never deployed. Removing both meta skill and slow update drops SpreadsheetBench 77.5→55.0 (−22.5, the largest ablation degradation).
- **`best_skill.md` is compact and cheap**: final skills span 379 tokens (LiveMath) to 1,995 tokens (SpreadsheetBench), median ~920, assembled from only **1–4 accepted edits** (median 2.5). LiveMath's +29.3 and OfficeQA's +39.0 gains each arose from a *single* accepted edit — direct evidence the gate does the work. Deployment adds **zero optimizer calls, zero weight updates**.
- **52/52 best-or-tied** across GPT-5.5, GPT-5.4, GPT-5.4-mini, GPT-5.4-nano, GPT-5.2, Qwen3.5-4B, and Qwen3.6-35B-A3B (direct chat) plus GPT-5.5 under Codex and Claude Code. Average per-model improvement ≈ **+17.6 points**; small/weak models gain most (GPT-5.4-nano triples on ALFWorld, doubles on DocVQA).
- **GPT-5.5 direct-chat per-benchmark gains over no skill**: SearchQA 77.7→87.3 (+9.6), SpreadsheetBench 41.8→80.7 (+38.9), OfficeQA 33.1→72.1 (+39.0), DocVQA 78.8→91.2 (+12.4), LiveMath 37.6→66.9 (+29.3), ALFWorld 83.6→95.5 (+11.9). Procedural benchmarks (spreadsheets, office docs, math) gain the most.
- **Harness-agnostic deployment**: a lightweight adapter injects the same `best_skill.md` into direct QA, spreadsheet execution, document/multimodal QA, embodied ALFWorld, and Codex/Claude Code loops. The single shared file format is what enables the cross-harness transfer experiments.
- **Transfer (Section 4.3)**: cross-model (all positive; one row beats its in-domain reference), cross-harness (SpreadsheetBench Codex→Claude Code +59.7), cross-benchmark (OlympiadBench→Omni-MATH +1.3 to +3.7). A stronger offline optimizer raises the deployed skill's quality monotonically *without* raising inference cost; a target-matched optimizer still recovers 56–74% of the gain (so this is not teacher→student distillation).
- **Learned rules are procedural, not instance-specific**: e.g. SpreadsheetBench — "Inspect workbook structure and formulas, then write evaluated static values across the full requested target range instead of relying on Excel recalculation." They read like rules a practitioner would write after a day with the benchmark, but are produced and validated edit-by-edit on held-out data.

## Entities Mentioned

- [[microsoft]] — Affiliation 1; home of corresponding author Yifan Yang. (See also [[microsoft-research]].)
- [[shanghai-jiao-tong-university]] — Affiliation 2; home of co-corresponding author Xue Yang.
- [[claude-code]] — One of the two tool-execution harnesses evaluated (alongside the Codex harness), driven via the `claude` CLI.

## Concepts Discussed

- [[skill-optimization]] — The paradigm this paper introduces and names: training a skill document as an optimizable artifact.
- [[agent-skills]] — Skills as a portable natural-language adaptation layer for frozen agents; the object SkillOpt optimizes.
- [[skill-md-format]] — The `SKILL.md`/`best_skill.md` packaging the optimized artifact ships in.
- [[skill-acquisition]] — SkillOpt is a new mode of acquisition (feedback-driven text-space optimization) beyond human-authored / one-shot / RL-trained skills.
- [[self-improving-agent]] — SkillOpt improves the agent's external state under feedback rather than its weights or scaffold.
- [[agentic-context-engineering]] — The closest prompt-evolution sibling (and source of the TextGrad/GEPA baselines); see the Relation note below.
- [[agentic-harness-engineering]] — The harness-evolution sibling: evolves the *harness* via telemetry, where SkillOpt evolves a single *skill document*.
- [[reinforcement-learning]] — SkillOpt borrows RL/SGD vocabulary (rollout batches, learning-rate schedules, momentum-like meta updates) but performs no weight updates and no gradient computation.

## Notable Quotes

> "Agent skills today are hand-crafted, generated one-shot, or evolved through loosely controlled self-revision—none of which behaves like a deep-learning optimizer for the skill, and none of which reliably improves over its starting point under feedback. We argue the skill should instead be trained as the external state of a frozen agent, with the same discipline that makes weight-space optimization reproducible."

> "SkillOpt is, to our knowledge, the first systematic controllable text-space optimizer for agent skills: a separate optimizer model turns scored rollouts into bounded add/delete/replace edits on a single skill document, and an edit is accepted only when it strictly improves a held-out validation score."

> "The deep-learning analogy is operational rather than decorative. ... the textual learning rate and schedule control how far one skill version is allowed to move from the previous one; the held-out gate plays the role of validation; and the epoch-wise slow/meta update acts like a momentum term, carrying stable editing directions across epochs."

## Relation to ACE, SkillMOO, and harness evolution

> [!note] Where SkillOpt sits among text-artifact optimizers
> All three of these systems evolve a natural-language artifact via LLM-proposed edits gated on execution feedback, but they differ in *what* is optimized and *under what objective*.

- **vs. [[agentic-context-engineering|Agentic Context Engineering (ACE)]]** (Zhang et al., [[stanford-university]]/[[sambanova]]): Both evolve a text artifact through LLM edits, and SkillOpt cites ACE's siblings (GEPA, TextGrad) as baselines. The distinction: **ACE evolves a structured prompt *playbook*** — an itemized list of bullets with helpful/harmful counters, merged by *deterministic non-LLM* rules, designed to fight brevity bias and context collapse during in-context adaptation. **SkillOpt optimizes a single *skill document* as a trainable parameter** under an explicit deep-learning discipline: a textual learning-rate budget, a *strict held-out validation gate* (ACE's merge is deterministic, not validation-gated), a rejected-edit buffer, and epoch-wise meta updates. ACE's artifact tends to grow as an append-mostly playbook; SkillOpt's stays deliberately compact (300–2,000 tokens, 1–4 edits) and ships with zero deployment-time optimizer calls. In short: ACE = playbook-with-counters that accumulates; SkillOpt = single-objective text-space optimizer that gates every edit on held-out improvement.
- **vs. [[skillmoo-gong-2026|SkillMOO]]** (Gong et al., King's College London et al.): SkillMOO is the **multi-objective** sibling — it Pareto-optimizes agent skills for software engineering across competing objectives. SkillOpt is **single-objective** (one scalar held-out score gates each edit). They share the "skill-as-optimizable-artifact" premise but differ on objective cardinality.
- **vs. [[agentic-harness-engineering|Agentic Harness Engineering]]** (Lin et al., Fudan): the **harness-evolution** sibling. It self-evolves the agent *harness* (tools, scaffolding, observability) via telemetry; SkillOpt holds the harness fixed and evolves only the *skill document* the harness loads.

## References

_Original source: `sources/skillopt-yang-2026.md` · [arXiv:2605.23904](https://arxiv.org/abs/2605.23904)_
