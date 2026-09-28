---
title: King's College London
type: entity
created: 2026-09-29
updated: 2026-09-29
sources:
  - skillmoo-gong-2026.md
  - agentcoder-huang-2023.md
status: draft
importance: low
tags:
  - university
---

# King's College London

UK university (KCL). In this wiki it appears through Jie M. Zhang's software-engineering group, which studies LLM code generation and agent skills from a testing and optimization angle.

## In this wiki

- **[[skillmoo-gong-2026|SkillMOO]]** (2026) — primary affiliation (Gong, Cao, Twist, Zhang). Treats task-specific [[agent-skills|agent skill]] bundles as a multi-objective search, keeping a Pareto front of pass rate against inference cost. It is the cost-aware counterpart to [[skillopt-yang-2026|SkillOpt]]'s single-objective optimization. Both agents run on [[glm-5|GLM-5]].
- **[[agentcoder-huang-2023|AgentCoder]]** (2023) — Jie M. Zhang (KCL) co-authored with HKU, SJTU and Sussex. A multi-agent code-generation loop in which a test-designer agent and an executor iterate on a programmer agent's code. See [[agentcoder]].

The common thread is test-driven evaluation of code-writing LLMs: first through generated tests, later through skill optimization scored by verifiers.

## Related

- [[agent-skills]] · [[skill-optimization]] · [[agentcoder]] · [[code-generation]] · [[hku]] · [[shanghai-jiao-tong-university]]

## References

- [[skillmoo-gong-2026]] · [[agentcoder-huang-2023]]
