---
title: "SkillNet: Create, Evaluate, and Connect AI Skills"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - skillnet-liang-2026.md
arxiv_id: "2603.04448"
authors:
  - Yuan Liang
  - et al.
first_author: Yuan Liang
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# SkillNet: Create, Evaluate, and Connect AI Skills

[arXiv:2603.04448](https://arxiv.org/abs/2603.04448) — Yuan Liang et al. (Zhejiang University, Tongji, Southeast University, and a large multi-institution / industry consortium incl. Alibaba, Ant, Tencent), 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

AI agents can invoke tools and execute complex tasks, but their long-term progress is hindered by the lack of systematic **skill accumulation and transfer** — without a unified consolidation mechanism, agents "reinvent the wheel," rediscovering solutions in isolation rather than reusing prior strategies. SkillNet is an open infrastructure for creating, evaluating, and organizing AI skills at scale. It structures skills within a unified ontology that ingests skills from heterogeneous sources, establishes rich relational connections (similarity, composition, dependency), and supports multi-dimensional evaluation across five axes: Safety, Completeness, Executability, Maintainability, and Cost-awareness.

The release bundles a repository of 200,000+ skills (150,000+ curated), an interactive platform, and a Python toolkit for skill discovery/creation/analysis. On ALFWorld, WebShop, and ScienceWorld, equipping agents with SkillNet improves average rewards by ~40% and reduces execution steps by ~30% across multiple backbone models. The framing is explicitly Sutton's "era of experience": formalize skills as evolving, composable assets so agents move from transient episodic experience to durable, transferable mastery.

## Key points

- Unified skill ontology with relational structure: similarity, composition, and dependency edges form a "skill network."
- Five-axis evaluation rubric: Safety, Completeness, Executability, Maintainability, Cost-awareness.
- Scale: 200k+ total skills (150k+ curated), plus an interactive platform and a Python toolkit.
- Skills created from heterogeneous sources, not just hand-authored.
- Benchmarks: ALFWorld, WebShop, ScienceWorld — +~40% average reward, −~30% execution steps across backbones.
- Positions skills as durable, composable assets to escape per-task in-context reinvention.
- Open release: skillnet.openkg.cn and github.com/zjunlp/SkillNet.

## Concepts & entities

- [[agent-skills]] — directly an infrastructure for agent skills at scale.
- [[skill-library]] — the 200k+ skill repository is a large-scale skill library.
- [[skill-acquisition]] — creating skills from heterogeneous sources.
- [[skill-evaluation]] — the five-axis multi-dimensional rubric.
- [[experiential-learning]] — Sutton "era of experience" framing.
- [[knowledge-graph]] — ontology + typed relations between skills.
- [[zhejiang-university]] — lead affiliation.

## References

- [[agent-skills]], [[skill-library]], [[skill-evaluation]] — core concepts.
- [[voyager]] — skill-library-for-agents precursor.
- [[experiential-learning]] — broader framing.
