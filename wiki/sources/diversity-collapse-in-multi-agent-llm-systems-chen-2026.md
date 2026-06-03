---
title: "Diversity Collapse in Multi-Agent LLM Systems"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - diversity-collapse-in-multi-agent-llm-systems-chen-2026.md
arxiv_id: "2604.18005"
authors:
  - Nuo Chen
  - Yuzhe Yang
  - Yicheng Tong
  - Yufei He
  - Xueyi Zhang
  - Qingyun Zou
  - Qian Wang
  - Bingsheng He
first_author: Nuo Chen
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Diversity Collapse in Multi-Agent LLM Systems

*Surfaced via the alphaXiv / NLP-newsletter digest.*

[arXiv:2604.18005](https://arxiv.org/abs/2604.18005) — Chen et al. (NUS, CUHK-Shenzhen), 2026.

## Summary

Multi-agent systems (MAS) are widely assumed to broaden exploration in open-ended idea generation, but this paper presents a systematic empirical study showing the opposite often holds. Across three bottom-up levels — model intelligence, agent cognition, and system dynamics — the authors find that collaboration frequently *contracts* the solution space rather than expanding it. They name this failure mode **diversity collapse**, driven by **structural coupling**: the interaction structure itself, not model capacity, inadvertently narrows each agent's exploration and pushes the group toward premature convergence. The takeaway is that designing MAS for creative tasks requires deliberately preserving agent independence and disagreement.

## Key points

- **Compute efficiency paradox** (model level): stronger, highly aligned models yield diminishing marginal diversity despite higher per-sample quality — better models can make the collective *less* diverse.
- **Authority-driven suppression** (cognition level): groups with dominant "authority" agents suppress semantic diversity relative to junior-dominated groups.
- **Group-size scaling** shows diminishing returns; adding more agents does not reliably broaden the explored idea space.
- **Dense communication topologies** accelerate premature convergence; sparser interaction preserves exploration longer.
- Collapse arises **primarily from interaction structure**, not inherent model insufficiency — a structural rather than capability problem.
- Practical guidance: maintain independence and encourage disagreement when building MAS for ideation/creative work.

## Concepts & entities

- [[multi-agent]]
- [[llm-agent]]

## References

- [arXiv:2604.18005](https://arxiv.org/abs/2604.18005)
- Code: https://github.com/Xtra-Computing/MAS_Diversity
- Local source: `sources/diversity-collapse-in-multi-agent-llm-systems-chen-2026.md`
