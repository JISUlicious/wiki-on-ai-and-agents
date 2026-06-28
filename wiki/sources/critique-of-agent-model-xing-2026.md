---
title: "Critique of Agent Model"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - critique-of-agent-model-xing-2026.md
arxiv_id: "2606.23991"
authors:
  - Eric Xing
  - Mingkai Deng
  - Jinyu Hou
first_author: Eric Xing
year: 2026
venue: arXiv (cs.AI)
tags:
  - 2026
status: complete
importance: medium
---

# Critique of Agent Model

**Source file**: [[critique-of-agent-model-xing-2026]]
**arXiv**: [arXiv:2606.23991](https://arxiv.org/abs/2606.23991) (v1, 22 Jun 2026, cs.AI)
**Authors**: Eric Xing, Mingkai Deng, Jinyu Hou (Eric Xing and Mingkai Deng co-first)
**Affiliations**: Institute of Foundation Models, MBZUAI; School of Computer Science, Carnegie Mellon University

_Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-21–06-28)._

## Summary

A position paper asking the foundational questions: *what is an agent, and what constitutes agency?* The authors argue the word "agent" has become nearly useless as a technical term, applied indiscriminately to "coding agents," "AI co-scientists," and "agentic" tools, while simultaneously fueling speculative fears of runaway "machine agency." Drawing on Descartes' grounding of agency in independent thought ("Cogito, ergo sum") and on science-fiction portrayals of autonomous beings (e.g., Blade Runner's replicants), they survey the [[llm-agent|LLM-agent]] landscape and analyze agent architectures along **five dimensions: goal, identity, decision-making, self-regulation, and learning**. Their central distinction: genuine agency requires these structures to be **internalized within the system** rather than assembled through external scaffolding. This separates *agentic* systems — whose competence lives in engineered workflows, tools, and control loops — from *agentive* systems, whose capabilities (including social interaction) arise endogenously and can operate in the open world with true autonomy. Building on the analysis they sketch a **Goal-Identity-Configurator (GIC) architecture** for a general-purpose agent, combining hierarchical goal decomposition, identity evolution, simulative reasoning grounded in a separately trained world model, learned self-regulation, and self-directed learning. They close with notes on auditability, controllability, and safety for higher-autonomy "agentive" systems kept under human oversight.

## Key points

- **"Agent" has lost technical meaning** — the term is stretched across marketed tools and existential fears alike; the paper aims to clarify where automation ends and agency begins.
- **Five dimensions of agency** — goal, identity, decision-making, self-regulation, and learning; used as the analytical axes for surveying current agent architectures.
- **Agentic vs. agentive** — the load-bearing distinction: competence from *external scaffolding* (agentic) versus *endogenous, internalized* capability operating in the open world (agentive).
- **Philosophical + sci-fi grounding** — Descartes' "thinking defines existence" and Blade Runner replicants anchor the notion that agency is more than operational excellence.
- **Proposed GIC architecture** — Goal-Identity-Configurator: hierarchical goal decomposition, identity evolution, simulative reasoning over a separate world model, learned self-regulation, self-directed learning from real + simulated experience.
- **Safety framing** — discusses auditability, controllability, and oversight for more autonomous agentive systems.

## Concepts & entities

- [[llm-agent]] — the systems whose definition and "agency" the paper critiques and reframes
- [[multi-agent]] — touches on social interaction and inter-agent relationships as a marker of genuine (agentive) agency

## References

_Original source: `sources/critique-of-agent-model-xing-2026.md`_
_arXiv: [arXiv:2606.23991](https://arxiv.org/abs/2606.23991)_
