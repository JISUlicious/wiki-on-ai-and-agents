---
title: "From Memory to Skills: Evidence-Grounded Co-Evolution Governance for Long-Horizon LLM Agents"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - from-memory-to-skills-tang-2026.md
arxiv_id: "2607.16621"
year: 2026
venue: Preprint (comments say "Submitted into EMNLP'2026")
authors:
  - Bo Tang
  - Yang Zhang
  - Guomian Zhuang
  - Wenqiang Wei
  - Zhiyu Li
tags:
  - 2026
  - paper
status: complete
importance: high
---

# From Memory to Skills: Evidence-Grounded Co-Evolution Governance for Long-Horizon LLM Agents

**Bo Tang, Yang Zhang, Guomian Zhuang et al.** (MemTensor; University of Science and Technology of China; Hong Kong Polytechnic University; Fuzhou University; Xi'an Jiaotong University) — [arXiv:2607.16621](https://arxiv.org/abs/2607.16621). v1, 18 Jul 2026, cs.CL. Code released under `MemTensor/MemOS`.

## Summary

The paper's opening complaint is that existing agent memory systems "retrieve prior traces as **passive context** rather than converting them into executable capabilities." The concrete failure mode given: after repeatedly exploring the same repository, an agent still re-lists directories to find tests, config files, or build scripts, burning tokens re-deriving what it already learned. The proposed fix is **MSCE**, a **training-free** Memory–Skill Co-Evolution framework — it never updates the base LLM, only an external cognitive state `C = (M1, M2, M3, K)`.

MSCE's central design choice is that skills are **promoted from governed memory rather than distilled directly from raw trajectories**. Raw traces contain failed attempts, blind exploration, and environment-specific artifacts, so distilling straight from them yields brittle, over-specific procedures. Instead MSCE interposes a three-level hierarchy: **L1 trace memory** stores grounded step-level evidence `(state, action, observation, self-reflection, value)`; **L2 policy memory** induces recurring procedural patterns across episodes `(trigger, procedure, verification, boundary, supporting evidence)`; **L3 environmental cognition memory** abstracts *declarative* knowledge about how an environment is organized (entity/structure facts, action–response regularities, constraints), explicitly excluding imperative instructions. Only L2 policies that pass two gates — retained supporting evidence with positive estimated gain `G > θ_G`, plus *stability* (recent evidence fits the current trigger/procedure/boundary without forcing a rewrite) — crystallize into callable skills.

The second contribution is **reflection-weighted value backfilling**, which solves step-level credit assignment without training. Terminal feedback is sparse and delayed; per-step self-reflections are dense but noisy. MSCE couples them: `V(f_t) = α_t · R_i + (1 − α_t) · γ · V(f_{t+1})`, where `α_t` is a reflection weight scored by a prompted operator (faithful, concrete, causally informative, transferable reflections get high weight; empty or tautological ones get zero). High-α steps carry locally-informative evidence about the outcome; low-α steps mostly inherit value from later steps. These evidence-calibrated trace values are the single signal governing retrieval, policy induction, L3 abstraction, skill promotion, and revision.

## Key points

- **Setup**: all methods run on the OpenClaw v2026.5.7 runtime with identical tool access, interaction budgets, and task order. Backbone **GPT-5.2**; **GPT-4o** for auxiliary prompted operators (reflection scoring, L2 induction, L3 abstraction).
- **EvoAgentBench** (five domains: IR/BrowseComp-Plus, Math/OmniMath, SE/SWE-Bench, Code/LiveCodeBench, KW/GDPVal). MSCE is best or tied-best on Pass@1 in **all five** domains:

| Domain | Best non-MSCE baseline | MSCE Pass@1 | Δ |
|---|---|---|---|
| Information Retrieval | 21.54 (EverOS) | **26.15** | +4.61 |
| Mathematical Reasoning | 43.00 (EverOS) | **47.00** | +4.00 |
| Software Engineering | 38.46 (three-way tie) | **53.85** | +15.39 |
| Code Implementation | 61.54 (EvoSkill) | **61.54** (tie) | 0, but cost 3.9 → **2.0 turns** |
| Knowledge Work | 48.28 | **53.45** | +5.17 |

- **Gains are not bought with inference budget**: cost drops on IR, Math, Code and KW vs. the strongest baseline. The single exception is SE, where the +15.39 pp gain comes with cost rising 37.3 → 40.8 turns, attributed to the extra repository navigation, editing and testing that successful SWE solutions require.
- **LoCoMo** (long-term dialogue memory, GPT-4o judge): MSCE overall **61.23** judge / **49.89** F1 vs. best baseline SkillFlow-Evolve 59.22 / 48.71 — **+2.01** and **+1.18** points. Best on single-hop (75.98), multi-hop (47.87) and temporal (44.24); second on open-domain among evolving systems (28.13 vs. EverOS 29.17) — though the *vanilla* agent beats every memory system on open-domain (39.58), which the paper notes but does not dwell on. A much narrower margin than the EvoAgentBench results.
- **Ablations** (IR / Math / SE / Code / KW Pass@1 vs. full MSCE at 26.15 / 47.00 / 53.85 / 61.54 / 53.45): **flat memory** is catastrophic (10.77 / 31.00 / 34.62 / 56.41 / 37.93) — the hierarchy, not merely storing experience, is what works. Removing **skill crystallization** costs 6.15 / 8.00 / 11.54 / 7.69 / 8.62 pp; removing **reflection weighting** costs 4.61 / 7.00 / 7.70 / 5.13 / 6.90 pp; **w/o L3** and **w/o value calibration** are the mildest.
- **Cross-domain transfer**: initializing on a source domain's evolved memory + skills improves Pass@1 on **all six** transfer pairs tested, **+2.56 to +5.13 pp** (average **+3.93 pp**), with cost falling in four of six pairs (the two transfers *into* Code raise cost +4.6% and +8.1%).
- **Skill reliability** is a smoothed success rate `η = (n_pass + 1)/(n_trial + 2)`; skills move through **probationary → active → archived**. Lifecycle events: successful invocation reinforces, execution failure repairs, user rejection shrinks the boundary, counter-evidence revises, source-policy rewrite rebuilds, long inactivity or low reliability archives.
- **Grounding is enforced deterministically**, not by trust: a verifier checks each draft skill's schema, requires cited evidence IDs to come from the support set and declared tools from the whitelist actually observed in evidence traces, and runs two coverage tests so the draft cannot invent unsupported commands. Failing drafts are discarded rather than exposed as callable.
- Key thresholds: `n_min = 2` distinct episodes before an L2 policy is induced; `v_min = 0.1` minimum trace value for L2 association; `θ_G = 0` gain threshold for promotion; `γ = 0.9` backfill discount.

## Concepts & entities

- [[skill-acquisition]] — the paper's core act: acquiring skills by *promotion from governed memory* under evidence/gain/stability gates, rather than by distillation from raw trajectories or by RL.
- [[agent-skills]] — crystallized skills are packaged as skill cards with trigger, procedure, verification rule, boundary, evidence anchors, decision guidance and reliability.
- [[memory-management]] — the L1/L2/L3 hierarchy plus retire/archive lifecycle is a memory-governance scheme; L1 stores normalized evidence (truncated text, capped tool I/O) rather than unbounded raw observations, for storage and privacy bounds.
- [[episodic-memory-llm]] — L1 trace memory is the episodic layer that everything higher must remain linked to for auditability.
- [[skill-optimization]] — skills are continuously repaired, shrunk, revised or archived from post-deployment invocation outcomes.
- [[voyager]] — the antecedent skill-library idea; MSCE's contrast is that Voyager-style direct distillation from trajectories is what it deliberately avoids.
- [[reflexion]] — self-reflections are the dense per-step signal, but here they are *scored* for faithfulness and used as interpolation weights rather than fed back verbatim.
- [[self-improving-agent]] — self-evolution without weight updates; the base LLM is frozen throughout.
- [[agentic-context-engineering]] — the entire method is context engineering: what gets written into, retrieved from, and retired out of the agent's external state.

## References

- [arXiv:2607.16621](https://arxiv.org/abs/2607.16621)
- Source text: [[from-memory-to-skills-tang-2026]]
- Code: `https://github.com/MemTensor/MemOS`
