---
title: "Agent-as-a-Router: Agentic Model Routing for Coding Tasks"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - agent-as-a-router-zhou-2026.md
arxiv_id: "2606.22902"
authors:
  - Pengfei Zhou
  - Zhiwei Tang
  - Yixing Ma
  - Jiasheng Tang
  - Wangbo Zhao
  - Yang You
first_author: Pengfei Zhou
year: 2026
venue: arXiv (cs.AI)
tags:
  - 2026
status: complete
importance: medium
---

# Agent-as-a-Router: Agentic Model Routing for Coding Tasks

**Source file**: [[agent-as-a-router-zhou-2026]]
**arXiv**: [arXiv:2606.22902](https://arxiv.org/abs/2606.22902) (v2, 24 Jun 2026, cs.AI)
**Authors**: Pengfei Zhou, Zhiwei Tang, Yixing Ma, Jiasheng Tang, … Wangbo Zhao, Yang You
**Affiliations**: National University of Singapore; DAMO Academy, Alibaba Group; Hupan Lab; UC Berkeley; Zhejiang University; HKUST

_Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-21–06-28)._

## Summary

Users increasingly have access to many LLMs from different providers, each strong in distinct domains, none dominant — so routing each task to the most suitable model is critical for both quality and cost. This paper argues that existing routers treat routing as a **static, one-off classification problem**, and identifies their core bottleneck as an **information deficit**: the router simply lacks the right knowledge about which model is good at what. As evidence, augmenting a vanilla LLM router with **task-dimension-level performance statistics** yields a 15.3% relative gain — beating even a heuristic router built on the same dimension-level priors. Motivated by this, the authors propose **Agent-as-a-Router**, which reframes routing as a *dynamic* C-A-F loop (Context → Action → Feedback → Context) that accumulates execution-grounded experience during deployment to close the information gap. The framework is instantiated as **ACRouter** (Orchestrator + Verifier + Memory), and the paper introduces **CodeRouterBench**, ~10K coding task instances with verified scores from 8 frontier LLMs, enabling regret-based comparison on streaming tasks. ACRouter achieves the lowest cumulative regret in-distribution and generalizes to out-of-distribution agentic-programming tasks.

## Key points

- **Routing is dynamic, not static classification** — the paper recasts model routing as an online loop that learns from execution feedback rather than a one-shot classifier.
- **"Information deficit" is the bottleneck** — routers fail mainly because they lack task-dimension performance knowledge, not because of weak policy models.
- **Dimension-level stats give a 15.3% relative gain** — augmenting a vanilla LLM router with per-task-dimension performance statistics surpasses a heuristic router on the same priors.
- **C-A-F loop** — Context→Action→Feedback→Context; accumulates execution-grounded experience during deployment to actively close the information gap.
- **ACRouter = Orchestrator + Verifier + Memory** — the concrete instantiation of the framework.
- **CodeRouterBench** — ~10K coding tasks with verified scores from 8 frontier LLMs; supports regret-based, streaming router evaluation; ACRouter wins in- and out-of-distribution.

## Concepts & entities

- [[multi-agent]] — orchestrates a pool of frontier LLMs via learned, feedback-driven model routing for coding tasks

## References

_Original source: `sources/agent-as-a-router-zhou-2026.md`_
_arXiv: [arXiv:2606.22902](https://arxiv.org/abs/2606.22902)_
