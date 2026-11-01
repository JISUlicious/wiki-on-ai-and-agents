---
title: "Object-Oriented Harness: Evolving Agent Harnesses through Object-Oriented Design"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - object-oriented-harness-li-2026.md
arxiv_id: "2610.28380"
authors:
  - Gaotang Li
  - Bhavana Dalvi Mishra
  - Yifei Ming
  - Yanfei Chen
  - Zhongying CuiZhu
  - Lesly Miculicich
  - Long T. Le
  - Rui Meng
  - Chun-Liang Li
  - Hanghang Tong
  - Chen-Yu Lee
  - Tomas Pfister
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Object-Oriented Harness (OO-Harness)

**Gaotang Li, Bhavana Dalvi Mishra, Yifei Ming, Yanfei Chen, Zhongying CuiZhu, Lesly Miculicich, Long T. Le, Rui Meng, Chun-Liang Li, Hanghang Tong, Chen-Yu Lee, Tomas Pfister** ([[uiuc|UIUC]]; [[google|Google Cloud AI Research]]) — [arXiv:2610.28380](https://arxiv.org/abs/2610.28380), v1, cs.CL, 20 Oct 2026. Preprint. The work was done during Li's Google internship.

## Summary

Global harness optimizers such as [[meta-harness-lee-2026|Meta-Harness]] and [[agentic-harness-engineering-lin-2026|AHE]] pick **one harness for all tasks**, and edits that help one task group can hurt another. OO-Harness evolves a **class hierarchy** of harnesses instead.

Training proceeds in three steps:

1. Discovery tasks are clustered by the execution needs seen in base-harness trajectories.
2. An evolution agent (Claude Opus 4.8, up to 40 iterations) repeatedly picks a cluster and a parent class.
3. It writes a **subclass** that overrides or adds methods, and evaluates that subclass only on its cluster.

Design lessons and *routing lessons* accumulate along the way. At test time the base harness runs first. A router (Claude Sonnet 5) then reads the task, the base trajectory and the routing lessons, and either picks a specialized subclass or keeps the base result. The executor is gemini-3.5-flash throughout.

## Key points

- **Held-out results** (3 runs, mean ± sd; base → OO-Harness):
  - AutomationBench mean partial credit: **61.62 → 77.41**.
  - Workspace-Bench micro rubric pass: **25.45 → 39.48**.
  - Toolathlon-GYM PASS: **31.15 → 37.45**.

  OO-Harness has the highest primary metric on all three. Meta-Harness reaches 63.92, 29.56 and 33.45 on the same three, and AHE 64.18, 38.43 and 36.23.
- **The nearest rival is another specialized family.** Adaptive Auto-Harness (AAH), which routes among Git branches, scores 75.40, 38.62 and 34.32. Against AAH, OO-Harness's Workspace-Bench lead (+0.86) is well inside the run-to-run spread (±2.98–3.21).
- **More test-time sampling of the base harness does not help.** Parallel best-of-2 or best-of-5 and sequential self-refinement leave AutomationBench flat or lower (59.3–61.5).
- **Inheritance ablation** (Workspace-Bench Lite-100, one run per variant). Subclassing reaches 40.56%. Flat copying of earlier evolved code scores **4.96 points lower**, and resetting to base scores lower still.
- **Routing evidence.** Routing on task text alone gets 64.88 on AutomationBench. Adding the base trajectory raises it to 77.52, so most of the gain needs the observed base run.
- **Out of distribution** (Toolathlon-GYM train → Toolathlon test, disjoint app families). The router picks Base for every test task and keeps 59.18% PASS. Meta-Harness's global harness drops to 44.90%.
- Search is faster than Meta-Harness: 29.0 h vs 109.1 h.

> [!warning] Newsletter vs paper
> The numbers match Table 1. "Beats Meta-Harness" is true but understates the picture. The strongest baseline is AAH, which is within noise on Workspace-Bench and 2 points behind on AutomationBench. "Inheritance +4.96" is a single-run ablation on a 100-task subset. In the OOD test, OO-Harness "wins" only by **falling back to the unevolved base**, so it avoids harm but adds no gain.

## What it does not show

- **Extra inference is not costed.** Every test task pays for a full base rollout plus a router call (a different, stronger model) before the chosen subclass runs. The paper reports steps of only the final trajectory and gives no matched-cost comparison with that overhead.
- **One executor.** Only gemini-3.5-flash is used, and there is no transfer test to other executors.

## Concepts & entities

- [[agentic-harness-engineering]] · [[code-as-harness]] — hierarchical rather than global harness evolution.
- [[meta-harness-lee-2026]] · [[agentic-harness-engineering-lin-2026]] — the global baselines; compare [[rrsi-regularized-recursive-self-improvement-agent-harnesses-xia-2026]] on OOD regression of evolved harnesses.
- [[skill-routing]] · [[learned-orchestration]] — the router selects among harnesses as skill routers select among skills.
- [[gepa]] · [[self-improving-agent]] — cited self-evolution building blocks.
- [[harness2-recursive-agent-harnessing-xu-2026]] — Harness², from the same Google Cloud AI group, edits per task online; OO-Harness freezes a learned hierarchy.

## References

- [arXiv:2610.28380](https://arxiv.org/abs/2610.28380)
