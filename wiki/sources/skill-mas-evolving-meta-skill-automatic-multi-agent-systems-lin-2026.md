---
title: "Skill-MAS: Evolving Meta-Skill for Automatic Multi-Agent Systems"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - skill-mas-evolving-meta-skill-automatic-multi-agent-systems-lin-2026.md
arxiv_id: "2606.18837"
authors:
  - Hehai Lin
  - Qi Yang
  - Chengwei Qin
first_author: Hehai Lin
year: 2026
venue: arXiv preprint
tags: [2026]
status: complete
importance: medium
---

# Skill-MAS: Evolving Meta-Skill for Automatic Multi-Agent Systems

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-21–06-28).

## Summary

Automatic [[multi-agent]]-system (MAS) generation faces a dilemma between model capability and experience retention. Inference-time orchestration couples a frozen frontier Meta-agent with iterative search but is experience-agnostic — it repeats identical searches without learning from past failures. Training-time orchestration fine-tunes smaller (~7B) LLMs to internalize experience via gradient updates but is capped by those models' low capability ceiling and can't scale to proprietary >100B frontier models. Skill-MAS (Ant Group, HKUST-GZ) proposes a **third path** that decouples experience retention from parametric updates: high-level orchestration knowledge is treated as an **evolvable Meta-Skill** — structured natural-language guidance refined across tasks via hierarchical contrastive analysis — letting a frozen frontier Meta-agent progressively learn.

## Key points

- Frames orchestration as an **evolvable Meta-Skill**: a third path between frozen inference-time methods and training-time fine-tuning, **decoupling experience retention from weight updates**.
- Distills systemic experience into generalizable, strategy-level principles via **hierarchical contrastive analysis**, iteratively refining the skill (removing or rewriting ineffective guidance).
- Evaluated across **four** challenging benchmarks (deep research, expert-level math, multi-hop QA via BrowseCompPlus, real-world interactive via VitaBench) and **four** distinct LLMs as Meta-agent.
- Reports remarkable performance gains while maintaining a favorable **cost–performance trade-off** — no costly parametric updates.
- Connects the emerging "Skill" paradigm to automatic-MAS, positioning Skill-MAS as a better third path for scalable orchestration.

## Concepts & entities

- [[multi-agent]] — automatic generation/orchestration of multi-agent systems.
- [[agent-skills]] — the Meta-Skill as structured natural-language orchestration knowledge.
- [[skill-acquisition]] — iterative refinement of the Meta-Skill across tasks without gradient updates.

## References

- [arXiv:2606.18837](https://arxiv.org/abs/2606.18837) — Lin, Yang, Qin. "Skill-MAS: Evolving Meta-Skill for Automatic Multi-Agent Systems" (2026). Ant Group; HKUST (Guangzhou).
