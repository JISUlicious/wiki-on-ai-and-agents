---
title: "Beyond Domains: Reusing Web Skills via Transferable Interaction Patterns"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - beyond-domains-reusing-web-skills-he-2026.pdf
arxiv_id: "2606.17645"
authors:
  - Shiqi He
  - et al.
first_author: Shiqi He
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Beyond Domains: Reusing Web Skills via Transferable Interaction Patterns

*Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-14–06-21).*

## Summary

LLM web agents are typically deployed as tool callers that read a fresh page observation each turn and emit one low-level primitive action. As horizons grow, policy-facing LLM completions dominate latency and cost on benchmarks like Mind2Web and WebArena. Recent systems wrap repeated interaction fragments as reusable *web skills*, but prior skill libraries are triggered mainly by instruction similarity or coarse site metadata — yielding low skill reuse on held-out sites.

**SkillMigrator** makes induced web-agent skills reusable *across* sites by matching **page-layout structure** rather than specific element references or site/instruction metadata. Each induced skill is stored as a **Transferable Interaction Pattern (TIP)**: the skill paired with a structural sketch of the snapshot at induction time. At test time the agent retrieves TIPs by layout similarity and grounds their references on the live page. The rest of the stack is standard — accessibility-snapshot observations with stable references, and fixed tool calling over primitives plus skill invocations. The result is fewer expensive LLM actions at comparable task success.

## Key points

- Stores skills as **Transferable Interaction Patterns (TIPs)** — a skill plus a structural sketch of the page snapshot — keyed by **layout structure**, not instruction or site metadata.
- Retrieval is by **layout similarity**; the skill's element references are grounded on the live page at test time.
- Reduces the **average LLM-action count on successful trajectories by 8–10%** across both WebArena and Mind2Web at **matched success rate**.
- Targets the cost/latency bottleneck where low-level primitives lengthen horizons and inflate policy LLM completions.
- Improves cross-site skill reuse versus libraries triggered by instruction similarity or coarse site metadata.
- Built on standard accessibility-snapshot observations with stable references and fixed tool calling.

## Concepts & entities

- [[skill-acquisition]]
- [[computer-use-agent]]
- [[agent-skills]]

## References

- [arXiv:2606.17645](https://arxiv.org/abs/2606.17645)
