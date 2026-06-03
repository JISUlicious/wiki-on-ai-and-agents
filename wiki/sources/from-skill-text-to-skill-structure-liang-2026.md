---
title: "From Skill Text to Skill Structure: The Scheduling-Structural-Logical Representation for Agent Skills"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - from-skill-text-to-skill-structure-liang-2026.pdf
arxiv_id: "2604.24026"
authors:
  - Qiliang Liang
  - Hansi Wang
  - Zhong Liang
  - Yang Liu
first_author: Qiliang Liang
year: 2026
tags: [2026]
status: complete
importance: medium
---

# From Skill Text to Skill Structure: The Scheduling-Structural-Logical Representation for Agent Skills

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

LLM agents increasingly rely on reusable skills — capability packages combining instructions, control flow, constraints, and tool calls — but these are still represented as text-heavy `SKILL.md`-style documents whose machine-usable evidence stays buried in natural language. The paper argues this creates a *representation problem*: managing and using skills requires reasoning over invocation interfaces, execution structure, and side effects, yet these signals are entangled in one textual surface. Drawing on Schank and Abelson's cognitive-linguistics work (Memory Organization Packets, Script Theory, Conceptual Dependency), the authors introduce the **Scheduling-Structural-Logical (SSL)** representation — claimed as the first structured representation for agent skill artifacts.

## Key points

- SSL disentangles three layers: **skill-level scheduling** signals, **scene-level execution** structure, and **logic-level** action/resource-use evidence.
- Instantiated with an LLM-based normalizer that converts text skill docs into the structured form.
- Evaluated on two tasks — Skill Discovery and Risk Assessment.
- Skill Discovery: MRR@50 improves from 0.649 (text-only baseline) to 0.729.
- Risk Assessment: macro F1 improves from 0.409 to 0.509.
- Positioned as a practical step toward inspectable, reusable, operationally actionable skills — explicitly not a finished standard or end-to-end management system.

## Concepts & entities

[[skill-acquisition]] · [[agent-skills]] · [[skill-optimization]] · [[memory-management]] · [[llm-agent]]

## References

- [arXiv:2604.24026](https://arxiv.org/abs/2604.24026)
