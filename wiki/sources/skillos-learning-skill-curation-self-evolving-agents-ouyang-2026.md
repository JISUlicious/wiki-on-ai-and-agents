---
title: "SkillOS: Learning Skill Curation for Self-Evolving Agents"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - skillos-learning-skill-curation-self-evolving-agents-ouyang-2026.pdf
arxiv_id: "2605.06614"
authors:
  - Siru Ouyang
  - Jun Yan
  - Yanfei Chen
  - Rujun Han
  - Zifeng Wang
  - Chen-Yu Lee
first_author: Siru Ouyang
year: 2026
tags: [2026]
status: complete
importance: medium
---

# SkillOS: Learning Skill Curation for Self-Evolving Agents

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

SkillOS (UIUC, Google Cloud AI Research, MIT) is an experience-driven RL recipe for teaching LLM agents to *curate* reusable skills so they self-evolve across streaming tasks instead of solving each one from scratch. The architecture pairs a **frozen agent executor** (which retrieves and applies skills) with a **trainable skill curator** that edits an external SkillRepo (insert/update/delete) from accumulated experience. The central bottleneck the paper attacks is learning long-horizon curation policies from indirect, delayed feedback. To supply learning signal, SkillOS trains on grouped task streams organized by skill-relevant task dependencies: earlier trajectories update the SkillRepo and later related tasks evaluate those edits, with composite rewards attributing downstream executor outcomes back to curation decisions.

## Key points

- Decouples a frozen executor from a trainable curator; only curation is learned via RL.
- Curator maintains an external SkillRepo through insert/update/delete operations grounded in past trajectories.
- Grouped task streams (by skill dependency) let earlier edits be scored by later related tasks, creating credit signal.
- Composite rewards attribute delayed downstream executor feedback to specific curation actions.
- Beats memory-free and strong memory-based baselines on multi-turn agentic and single-turn reasoning tasks in effectiveness and efficiency.
- Learned curator generalizes across executor backbones and domains; SkillRepo develops richer structure and higher-level meta-skills over time.

## Concepts & entities

- [[skill-acquisition]]
- [[skill-optimization]]
- [[reinforcement-learning]]
- [[llm-agent]]
- [[procedural-memory]]

## References

- [arXiv:2605.06614](https://arxiv.org/abs/2605.06614)
