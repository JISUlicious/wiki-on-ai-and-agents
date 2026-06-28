---
title: "NatureBench: Can Coding Agents Match the Published SOTA of Nature-Family Papers?"
type: source
created: 2026-06-29
updated: 2026-06-29
sources:
  - naturebench-coding-agents-nature-family-papers-wang-2026.md
arxiv_id: "2606.24530"
authors:
  - Yuru Wang
  - Lejun Cheng
  - Yuxin Zuo
  - Kaiyan Zhang
  - Ning Ding
  - Bowen Zhou
first_author: Yuru Wang
year: 2026
venue: arXiv preprint
tags: [2026, benchmark]
status: complete
importance: medium
---

# NatureBench: Can Coding Agents Match the Published SOTA of Nature-Family Papers?

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-21–06-28).

## Summary

NatureBench (Horizon Research / Frontis.AI, Tsinghua University, Peking University) is a cross-discipline benchmark of **90 tasks** distilled from peer-reviewed Nature-family publications, designed to test whether AI coding agents can move beyond *reproduction* toward genuine *discovery* on real scientific problems. It is built on **NatureGym**, an automated pipeline that constructs a standardized, per-task **containerized environment** from each source paper — directly addressing the environment-fragmentation problem that has undermined the credibility of prior agent-on-research benchmarks. Agents are evaluated under a strict **web-search-disabled** protocol so they cannot simply retrieve the original method.

## Key points

- **90 tasks** distilled from peer-reviewed Nature-family papers; each runs in a standardized containerized environment via the **NatureGym** pipeline.
- **Web-search-disabled** protocol prevents agents from looking up the source method — isolating discovery from retrieval/reproduction.
- Across **ten** frontier agent configurations, the **strongest model surpasses SOTA on only 17.8%** of tasks (under the g > 0.1 effect-size criterion).
- Pathway analysis: agents succeed mainly through **methodological translation** (recasting scientific tasks as familiar supervised-prediction problems), not genuine scientific invention.
- Failures dominated by **wrong method choice** and **insufficient compute budget**, not task misunderstanding; benchmark, pipeline, and a public leaderboard with maintainer-side reproduction released.

## Concepts & entities

- [[agent-benchmark]] — a discovery-oriented benchmark for AI coding/research agents.
- [[code-generation]] — coding agents tasked with implementing methods to match published SOTA.

## References

- [arXiv:2606.24530](https://arxiv.org/abs/2606.24530) — Wang, Cheng, Zuo, Zhang, Ding, Zhou (Horizon Research / Frontis.AI; Tsinghua University; Peking University). "NatureBench: Can Coding Agents Match the Published SOTA of Nature-Family Papers?" (2026).
