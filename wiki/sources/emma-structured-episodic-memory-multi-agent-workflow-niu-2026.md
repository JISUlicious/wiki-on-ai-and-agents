---
title: "EMMA: Structured Episodic Memory for Multi-Agent Workflow Synthesis"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - emma-structured-episodic-memory-multi-agent-workflow-niu-2026.md
arxiv_id: "2610.35966"
authors:
  - Ruijia Niu
  - Kaiwen Liu
  - Rose Yu
  - Kristina Klinkner
year: 2026
venue: arXiv preprint
tags: [2026, paper]
status: complete
importance: medium
---

# EMMA: Structured Episodic Memory for Multi-Agent Workflow Synthesis

**Source:** Ruijia Niu, Kaiwen Liu, Rose Yu, Kristina Klinkner (UC San Diego, [[amazon|Amazon]]) — [arXiv:2610.35966](https://arxiv.org/abs/2610.35966), v1 25 Oct 2026. Surfaced via the NLP newsletter (2026-11-01).

## Summary

EMMA adapts a [[multi-agent]] system to a specialized domain without touching weights. It treats the **system's configuration itself as [[episodic-memory-llm|episodic memory]]**, learned from training-task failures. The paper's diagnosis is that existing multi-agent evolution methods ([[adas|ADAS]], EvoMAS, AFlow) are *ill-scoped*: the optimizing backbone decides where and how much to change the system, so what gets learned reflects the backbone's habits rather than the failures. In their Figure 1, Claude Opus 5 mostly accumulates skills (18 on one benchmark), while DeepSeek v3.2 spawns agents (114 over 83 training tasks).

The memory has three levels of decreasing generality. **Controller memory** holds routing and decomposition rules, which the controller picks from an index rather than by similarity search. The **agent pool** grows from a single generalist; every specialist's prompt is the generalist prompt plus a role-specific part, so general fixes reach all agents. The **skill pool** holds short per-sub-problem procedures that the controller assigns per step. Execution follows the Magentic-One pattern: plan, dispatch one step at a time, read the output, replan. A step that overflows its agent's context or round budget is split into smaller steps (overflow decomposition), not treated as a failure.

## Key points

- **Scoped reflection.** On each failed training attempt a reflector sees the full trajectory and then localizes, classifies, generalizes and minimizes. It assigns the failure to one of five scopes (execution slip, agent behavior, procedure, missing capability, routing), each with a fixed set of allowed actions, and proposes exactly one edit.
- **Backbone-independent guards** decide whether an edit is accepted: scope–action compatibility, prompt preservation, bounded pool sizes, no answer leakage, and a forced change of strategy after repeated failed edits. One-off slips never enter durable memory, and an edit is kept only if it is re-tested.
- **Results** (KramaBench, [[appworld|AppWorld]], BBEH-mini; backbones Claude Opus 5, Sonnet 5, DeepSeek v3.2, [[glm-5|GLM-5]], Kimi-K2.5; baselines [[agentic-context-engineering|ACE]], [[gepa|GEPA]], EvoMAS, ADAS). EMMA has the best mean on every benchmark: +7.0 points on KramaBench (0.654 vs GEPA 0.584), +5.8 on BBEH, and +11.9 TGC / +17.4 SGC on AppWorld normal over ACE. It ranks first in 17 of 20 settings, and the gain reaches up to 32.5% over the strongest baseline.
- **Backbone robustness.** EMMA's weakest backbone beats every baseline's weakest backbone. Its margin grows on open-weight models (e.g. AppWorld challenge: +0.2 TGC on Opus 5, but +15.4 and +16.6 on GLM-5 and DeepSeek). Search-based methods are the most backbone-sensitive.
- **Efficiency.** 17–38% fewer training executions than the most efficient baseline, because only failed tasks are re-run.

## Concepts & entities

- [[episodic-memory-llm]] · [[procedural-memory]] · [[multi-agent-memory]] · [[memory-management]] — configuration-as-memory across three levels.
- [[adas]] · [[gepa]] · [[agentic-context-engineering]] · [[prompt-optimization]] — baselines; EMMA adds explicit update ownership.
- [[reflexion]] · [[self-improving-agent]] · [[skill-library]] · [[subagents]] — reflection-driven, weight-free adaptation.
- [[appworld]] · [[amazon]] · [[glm-5]]

## References

- [arXiv:2610.35966](https://arxiv.org/abs/2610.35966)
