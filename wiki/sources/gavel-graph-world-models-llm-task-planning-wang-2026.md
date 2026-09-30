---
title: "GAVEL: Graph World Models for Verified and Efficient Long-Horizon LLM Task Planning"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - gavel-graph-world-models-llm-task-planning-wang-2026.md
arxiv_id: "2609.19315"
authors:
  - Ruiyang Wang
  - Hao-Lun Hsu
  - Swarajh Mehta
  - Jiwoo Kim
  - Zhihao Dou
  - Miroslav Pajic
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# GAVEL: Graph World Models for Verified and Efficient Long-Horizon LLM Task Planning

**Ruiyang Wang, Hao-Lun Hsu, Swarajh Mehta, Jiwoo Kim, Zhihao Dou, Miroslav Pajic.** No affiliation is printed in the arXiv text. [arXiv:2609.19315](https://arxiv.org/abs/2609.19315), cs.RO, 16 Sep 2026. Preprint.

## Summary

LLM task plans for robots skip preconditions, violate embodiment constraints and cope poorly with partial observability. GAVEL keeps the LLM as the *semantic* planner but runs every plan through an explicit **graph [[world-model]]**. This is a typed scene graph (rooms, objects, robot; relations such as `inside` and `on_top`; flags such as `open` and `washed`) with **hand-specified** preconditions and effects for nine BEHAVIOR-1K primitives. The graph rolls the plan forward and checks applicability, goal predicates and trace-level safety, such as closing what was opened and switching off appliances. It **repairs failures whose fix follows from the action model by itself**: a failed grasp inserts `NavigateTo`, and an object in a closed container inserts navigate plus `Open`. Only failures that need semantic reasoning go back to the LLM, within a budget of T = 5 re-queries. Under partial observability, a Relational Semantic Network (bge-small plus an MLP; AUC 0.904) keeps **full room-location distributions** for unseen objects. These are turned into expected search costs, and the remaining subtasks are re-ordered by exact permutation search (N ≤ 5) after each finished task.

## Key points

- **Single long-horizon tasks (Table I; 100 tasks, 10 scenes, 5 seeds).** With Qwen3-8B, llm-only scores 41.2%, SayPlan-style feedback 76.4% (2.7 calls), repair-only gavel-basic 76.2% (1 call) and full GAVEL **91.8%** (1.5 calls). With Qwen3-4B the figures are 21.8 → 55.4 → 67.7 → **88.8%**, and repair-only beats feedback-only by 12.3 points at a third of the calls.
- **Multi-task under partial observability (Table II; 500 instructions, 2–5 tasks, Qwen3-8B).** llm-only 19.9%, EPoG-style 60.2%, SayPlan-style 75.6%, GAVEL **92.6%**. EPoG's failures are exactly the instructions that need an appliance-induced state ("wash the plate"). Of GAVEL's 38 residual failures, 21 come from object extraction, 5 from unexecutable plans and 9 from unmet goals.
- **Scaling does not replace it (Table III; a 100-instruction subset).** Used alone, GPT-5.6 Sol scores 24.6% and Claude Sonnet 5 38.6%. With GAVEL they reach **99.2% / 99.4%**, and **Qwen3-4B + GAVEL reaches 75.2%**, beating both hosted models on their own. The world model adds almost no planning time for the hosted models (16.5 s → 16.5 s, 18.0 s → 18.2 s).
- **Belief reasoning.** Distance falls from 82.45 m (map-collapse) to 79.69 m (full distribution, fixed order) to **78.01 m** (online re-ordering). The oracle reaches 56.17 m. The most-likely room is right only 47% of the time on 832 held-out queries.

> [!note] Abstract vs body
> The abstract says distributional belief reasoning cuts travel "approximately 5.4% compared with a static variant". In the body, 5.4% (4.45 m) is measured against **gavel-map**, which collapses each belief to its most likely room. Against **gavel-static**, which keeps the distribution with a fixed order, online re-ordering saves only 1.68 m (about 2.1%). All three variants have identical success (92.6%).

## Concepts & entities

- [[world-model]] — note the stretch: that page defines a world model as *learned*, whereas GAVEL's transition model is a hand-written symbolic action model. Only the location beliefs are learned.
- [[planning]] · [[plan-execute-verify-loop]] · [[verifier]] — this is a verify-and-repair loop in which the verifier also repairs.
- [[embodied-agent]] · [[harness-vs-model-attribution]] — Table III is a same-harness, four-model test in which the harness contributes more than the model scale.
- [[structuredagent-andor-tree-planning-lobo-2026]] · [[reliability-limits-llm-multi-agent-planning-ao-2026]] — other 2026 structured-planning results.

## What it does not show

- Evaluation runs in a **2-D symbolic executor**. It was validated against OmniGibson only on the 100 single-task instructions (outcomes agree on every one), not on the 500 multi-task ones, and there is no low-level manipulation or real robot.
- The action semantics and safety rules are hand-authored for nine primitives. How much engineering a new domain would need is not measured.
- Multi-task instructions are independent by construction (disjoint goal objects), which makes ordering a cost question, not a feasibility one.

## References

- [arXiv:2609.19315](https://arxiv.org/abs/2609.19315)
