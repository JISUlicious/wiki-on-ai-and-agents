---
title: "Sakana Fugu Technical Report"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - sakana-fugu-sakana-ai-2026.md
arxiv_id: "2606.21228"
authors:
  - Fugu Team (Sakana AI)
first_author: Sakana AI
year: 2026
venue: arXiv (technical report)
tags:
  - 2026
status: complete
importance: medium
---

# Sakana Fugu Technical Report

**Source file**: [[sakana-fugu-sakana-ai-2026]]
**arXiv**: [arXiv:2606.21228](https://arxiv.org/abs/2606.21228) (2026-06-24)
**Authors**: Fugu Team, [[sakana-ai|Sakana AI]] (cite as "Sakana AI (2026)")
**Project**: https://sakana.ai/fugu

_Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-21–06-28)._

## Summary

Fugu is a family of **orchestrator models** from [[sakana-ai|Sakana AI]] — language models trained to read a user query and *dynamically devise the agentic scaffold* (which worker agents, in what structure) needed to solve it, rather than executing the task themselves. The motivation is that frontier LLMs from different providers increasingly specialize in distinct domains (e.g., one family strong at software engineering, another at reasoning), so the next gain comes from combining their specializations into a single collectively intelligent system. Fugu treats **orchestration as a new scaling axis** complementary to building ever-larger models: by routing and composing a team of frontier "worker" agents through learned, query-adaptive scaffolds, it reaches performance beyond any individual worker. Two models are released — **Fugu** (balances quality and latency; selects a single worker per input for everyday use) and **Fugu-Ultra** (maximizes answer quality on the hardest problems). The report claims state-of-the-art results among publicly accessible models across SWE-Bench Pro, Terminal Bench, LiveCodeBench, GPQA-Diamond, Humanity's Last Exam, and CharXiv Reasoning.

## Key points

- **Learned orchestration, not static routing** — Fugu is itself an LM that interprets a query and constructs an agentic scaffold on the fly, decomposing into natural-language subtasks each assigned to a worker agent from the pool.
- **Three-part training paradigm** — large-scale fine-tuning, evolutionary algorithms (sep-CMA-ES, following the Trinity optimization approach), and reinforcement learning over end-to-end agentic trajectories.
- **End-to-end trajectory training** — collects real multi-turn coding-assistant trajectories (Claude Code, Codex, OpenCode) so Fugu learns *practical* worker capability inside a tool-using harness, not just isolated benchmark scores.
- **Two release variants** — Fugu (single-worker, low latency) vs. Fugu-Ultra (quality-first on the hardest tasks).
- **Orchestration as a scaling axis** — new workers can be added to the pool without retraining the orchestrator; argues frontier progress need not depend solely on the largest training runs.
- **Broad SOTA claims** — best public results across SWE-Bench Pro, Terminal Bench, LiveCodeBench, GPQA-Diamond, Humanity's Last Exam, and CharXiv Reasoning.

## Concepts & entities

- [[sakana-ai]] — author/organization; releases Fugu and Fugu-Ultra
- [[multi-agent]] — Fugu orchestrates a team of frontier worker agents into a collectively intelligent system

## References

_Original source: `sources/sakana-fugu-sakana-ai-2026.md`_
_arXiv: [arXiv:2606.21228](https://arxiv.org/abs/2606.21228)_
