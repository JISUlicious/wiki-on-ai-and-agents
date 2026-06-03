---
title: "From Skills to Talent: Organising Heterogeneous Agents as a Real-World Company"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - from-skills-to-talent-organising-heterogeneous-agents-yu-2026.pdf
arxiv_id: "2604.22446"
authors:
  - Zhengxu Yu
  - Yu Fu
  - Zhiyuan He
  - Yuxuan Huang
  - Lee Ka Yiu
  - Meng Fang
  - Weilin Luo
  - Jun Wang
first_author: Zhengxu Yu
year: 2026
tags: [2026]
status: complete
importance: medium
---

# From Skills to Talent: Organising Heterogeneous Agents as a Real-World Company

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

OneManCompany (OMC) introduces an organisational layer for multi-agent systems, arguing that current systems are constrained by fixed team structures, tightly coupled coordination, and session-bound learning. OMC encapsulates skills, tools, and runtime configs into portable agent identities called **Talents**, orchestrated through typed organisational interfaces that abstract over heterogeneous backends. A community-driven **Talent Market** enables on-demand recruitment so an organisation can close capability gaps and reconfigure itself mid-execution. Decision-making runs through an **Explore-Execute-Review (E²R)** tree search that unifies planning, execution, and evaluation in one hierarchical loop with formal guarantees on termination and deadlock freedom.

## Key points

- Decouples *what an agent knows* (skills) from *how a workforce is assembled and governed* (the organisational layer) — the central gap the paper claims existing MAS frameworks miss.
- **Talent-Container architecture**: per-employee profiles track skills, performance, and configuration, giving each agent a portable identity.
- **E²R tree search**: tasks decomposed top-down into accountable units; execution outcomes aggregated bottom-up to drive systematic review and refinement.
- Organisation-level evolution via editable workflow SOPs and "company culture" rules that persist across projects, enabling self-improvement beyond a single session.
- Achieves 84.67% success on PRDBench, surpassing prior SOTA by 15.48 percentage points, with cross-domain case studies showing generality.
- Framed as a shift from static, pre-configured pipelines to self-organising, self-improving AI organisations.

## Concepts & entities

[[multi-agent]] · [[skill-acquisition]] · [[agent-skills]] · [[skill-optimization]] · [[self-improving-agent]] · [[llm-agent]]

## References

- [arXiv:2604.22446](https://arxiv.org/abs/2604.22446)
- Project homepage: https://one-man-company.com
