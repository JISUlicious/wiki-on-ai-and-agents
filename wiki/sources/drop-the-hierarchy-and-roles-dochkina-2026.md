---
title: "Drop the Hierarchy and Roles: How Self-Organizing LLM Agents Outperform Designed Structures"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - drop-the-hierarchy-and-roles-dochkina-2026.md
arxiv_id: "2603.28990"
authors:
  - Victoria Dochkina
first_author: Victoria Dochkina
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Drop the Hierarchy and Roles: How Self-Organizing LLM Agents Outperform Designed Structures

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

This paper reports a 25,000-task computational experiment — the largest to date — comparing coordination architectures in multi-agent LLM systems across 8 models, 4–256 agents, and 8 protocols. The headline result is the *endogeneity paradox*: a hybrid "Sequential" protocol, in which agent ordering is fixed but role selection is autonomous, outperforms both centralized coordination (+14%, p < 0.001) and fully autonomous protocols (+44%, Cohen's d = 1.86). Effective self-organization requires both a capable model and the right protocol — neither alone suffices, and models below a capability threshold show a reversal where rigid structure beats autonomy.

The system scales sub-linearly out to 256 agents and exhibits emergent organizational properties: dynamic role invention (5,006 unique roles emerging from 8 agents), voluntary self-abstention from out-of-competence tasks, and spontaneous hierarchy formation. Results reproduce across closed- and open-source models, with open-source models reaching 95% of the quality at 24x lower cost. The framing: agents need a mission, a communication protocol, and a sufficiently capable model to self-organize — and notably *not* a pre-assigned role.

## Key points

- Endogeneity paradox: hybrid protocol (fixed ordering, autonomous role selection) beats centralized (+14%) and fully autonomous (+44%) coordination.
- Self-organization needs *both* a capable model and the right protocol; below a capability threshold, rigid designed structure outperforms autonomy (a reversal).
- Emergent behaviors: 5,006 unique roles invented by 8 agents, voluntary self-abstention, and spontaneous hierarchy formation.
- Scales sub-linearly to 256 agents (p = 0.61 on the scaling test).
- Open-source models reach ~95% of closed-source quality at 24x lower cost.
- Contrasts "vertical self-improvement" (smarter individual agents) against "horizontal" coordination-architecture design.

## Concepts & entities

- [[multi-agent]] — central topic: self-organizing vs. designed coordination structures
- [[llm-agent]], [[reasoning]], [[planning]]
- [[emergent-abilities]] (emergent role invention, hierarchy formation)
- [[scaling-laws]] (sub-linear agent-count scaling)
- coordination protocols, organizational design (concept neighbors)
- Moscow Institute of Physics and Technology (MIPT)

## References

- [arXiv:2603.28990](https://arxiv.org/abs/2603.28990) — Dochkina, 2026 (v1, 30 Mar 2026).
