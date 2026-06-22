---
title: "Can LLM Agents Infer World Models? Evidence from Agentic Automata Learning"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - can-llm-agents-infer-world-models-menaged-2026.md
arxiv_id: "2606.16576"
authors:
  - Reef Menaged
  - et al.
first_author: Reef Menaged
year: 2026
venue: arXiv preprint (under review)
tags:
  - 2026
status: complete
importance: medium
---

# Can LLM Agents Infer World Models? Evidence from Agentic Automata Learning

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-14–06-21).

## Summary

This paper casts **world-model inference as agentic automata learning** to test how well tool-calling LLM agents can uncover hidden environments through interaction ([[world-model]]). The agent must identify a hidden deterministic finite automaton (DFA) by interacting with an oracle via two query types: **membership queries** ("Does this string belong to the target language?") and **equivalence queries** ("Is this the target DFA?"). Because the ground-truth DFA is known, this yields a scalable, gradeable testbed with controlled task complexity, measurable interaction efficiency, and strong reference baselines (classic automata-learning algorithms such as L*).

Evaluating state-of-the-art LLMs, the authors find performance **drops sharply as DFA size grows**. Reasoning models markedly outperform non-reasoning models, but trajectory analyses reveal recurring failures in query planning, evidence integration, and hypothesis construction. Overall, current LLM agents can sometimes perform non-trivial interactive discovery, yet remain far less robust and efficient than classic automata-learning algorithms.

## Key points

- **Agentic automata learning:** infer a hidden DFA via membership + equivalence queries against an oracle — a proxy for inferring an environment's world model through interaction.
- **Gradeable, scalable testbed** ([[agent-benchmark]]) with controlled complexity (DFA size), measurable interaction efficiency, and classic-algorithm baselines.
- **Performance drops sharply as DFA size increases** — agents struggle to scale interactive discovery.
- **Reasoning models clearly beat non-reasoning models**, but still fall short of classic algorithms.
- **Recurring failure modes** identified from trajectories: query planning, evidence integration, and hypothesis construction.
- **Conclusion:** LLM agents can occasionally do non-trivial interactive discovery but are far less robust and sample-efficient than classic automata learners.

## Concepts & entities

- [[world-model]] — inferring a hidden environment model is framed as automata learning.
- [[agent-benchmark]] — the DFA-inference setup is a controlled, gradeable agent testbed.

## References

- [arXiv:2606.16576](https://arxiv.org/abs/2606.16576)
- Source PDF: `can-llm-agents-infer-world-models-menaged-2026.pdf`
