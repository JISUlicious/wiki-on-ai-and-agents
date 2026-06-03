---
title: "WebXSkill: Skill Learning for Autonomous Web Agents"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - webxskill-skill-learning-web-agents-wang-2026.md
arxiv_id: "2604.13318"
authors:
  - Zhaoyang Wang
  - Qianhui Wu
  - Xuchao Zhang
  - Chaoyun Zhang
  - Wenlin Yao
  - Baolin Peng
  - Jianfeng Gao
  - Huaxiu Yao
first_author: Zhaoyang Wang
year: 2026
tags: [2026]
status: complete
importance: medium
---

# WebXSkill: Skill Learning for Autonomous Web Agents

*Surfaced via the alphaXiv / NLP-newsletter digest.*

## Summary

WebXSkill is a framework that lets LLM-powered web agents learn and reuse **executable skills** for long-horizon browser tasks. The authors identify a "grounding gap" in prior skill formulations: textual workflow skills give natural-language guidance but cannot be executed directly, while code-based skills are executable but opaque to the agent, offering no step-level understanding for error recovery. WebXSkill bridges this by pairing a parameterized action program with step-level natural-language guidance, supporting both direct execution and agent-driven adaptation. On WebArena and WebVoyager it improves task success rate by up to 9.8 and 12.9 points over the baseline.

## Key points

- Targets the brittleness of web agents on multi-page workflows, where procedural knowledge from successful trajectories is discarded and re-derived each time.
- Three-stage pipeline: **skill extraction** mines reusable action subsequences from synthetic agent trajectories and abstracts them into parameterized skills; **skill organization** indexes them into a URL-based graph for context-aware retrieval; **skill deployment** exposes two modes.
- Two deployment modes: **grounded mode** for fully automated multi-step execution, and **guided mode** where skills act as step-by-step instructions the agent follows with its native planning.
- Executable skills = parameterized action program + step-level NL guidance, giving both executability and interpretability.
- Evaluated on WebArena (+9.8 pts) and WebVoyager (+12.9 pts); code released at github.com/aiming-lab/WebXSkill.
- Work from UNC Chapel Hill and Microsoft.

## Concepts & entities

- [[skill-acquisition]]
- [[skill-library]]
- [[agent-skills]]
- [[gui-agent]]
- [[agentic-search]]
- [[procedural-memory]]
- [[microsoft]]

## References

- [arXiv:2604.13318](https://arxiv.org/abs/2604.13318)
- Source text: `sources/webxskill-skill-learning-web-agents-wang-2026.md`
