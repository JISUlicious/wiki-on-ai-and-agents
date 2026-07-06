---
title: "Generative Skill Composition for LLM Agents"
type: source
created: 2026-07-06
updated: 2026-07-06
sources:
  - generative-skill-composition.pdf
arxiv_id: "2606.32025"
authors:
  - Xinyu Zhao
  - Zhen Tan
  - Vaishnav Tadiparthi
  - Nakul Agarwal
  - Kwonjoon Lee
  - Ehsan Moradi Pari
  - Hossein Nourkhiz Mahjoub
  - Tianlong Chen
first_author: Xinyu Zhao
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Generative Skill Composition for LLM Agents

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-28–07-05).

## Summary

As LLM agents accumulate large libraries of reusable skills (modular packages of procedural knowledge — e.g., setting up a sandbox, running a test suite, refactoring across files), *selecting* the right skills becomes the central bottleneck. The paper argues existing approaches — exposing the agent to the whole collection, or embedding/LLM-reranker retrieval — miss the structural nature of the problem. It reframes skill selection as **structured skill composition**: a joint decision over *which* skills, *how many*, and *in what order*, three dimensions that cannot be decoupled. Their method, **SkillComposer**, casts this as task-conditioned skill-sequence prediction via a constrained autoregressive decoder over skill identifiers, so subset, count, and order emerge jointly from a single decoding pass.

## Key points

- Formalizes **structured skill composition**: given a task and skill library, predict an executable skill *plan* that jointly specifies activated subset, count, and execution order — versus treating selection as embedding-retrieval ranking.
- **SkillComposer** uses a constrained autoregressive decoder over skill IDs; dependencies between successive skills are captured naturally as a sequence-generation problem rather than independent scoring.
- Training set of task–composition pairs is built from a real, human-curated skill library.
- Evaluated on composition quality (held-out set) and downstream task success on **SkillsBench** across two production coding agents.
- On {GPT-5.2-Codex, Gemini-3-Pro-Preview}, raises pass rate by {+23.1, +18.2} pp over the no-skill baseline, surpassing top-3 retrieval and matching the gold-skill retrieval upper bound at lower prompt-token cost.

## Concepts & entities

Directly about [[agent-skills]] and their reuse; the composition/plan framing connects to [[learned-orchestration]] (choosing and sequencing capabilities) and to [[skill-acquisition]] (building and leveraging a growing skill library).

## References

- [arXiv:2606.32025](https://arxiv.org/abs/2606.32025) — "Generative Skill Composition for LLM Agents", Zhao et al. (UNC Chapel Hill / ASU / Honda Research Institute USA), 2026. Project page: https://skill-composer.github.io/
