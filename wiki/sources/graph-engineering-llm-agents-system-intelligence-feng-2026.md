---
title: "Graph Engineering in the Era of LLM Agents: From Individual Intelligence to System Intelligence"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - graph-engineering-llm-agents-system-intelligence-feng-2026.md
arxiv_id: "2608.21156"
authors:
  - Yuyuan Feng
  - Zhishang Xiang
  - Chaobin Yang
  - Qichao Ma
  - Qinggang Zhang
  - Yi Chang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Graph Engineering in the Era of LLM Agents: From Individual Intelligence to System Intelligence

**Yuyuan Feng, Zhishang Xiang, Chaobin Yang, Qichao Ma** (equal contribution) *et al.*, 35 authors. Project lead is Qinggang Zhang and corresponding authors include Yi Chang; the contact email and resource repository (DEEP-JLU) point to Jilin University. [arXiv:2608.21156](https://arxiv.org/abs/2608.21156), v2, cs.IR, 26 Aug 2026 (v1 21 Aug). Survey; resource list at github.com/DEEP-JLU/Awesome-Graph-Engineering.

## Summary

The survey describes four successive engineering paradigms for LLMs:

- [[prompt-engineering|Prompt Engineering]]: eliciting what the model can do;
- [[context-engineering|Context Engineering]]: managing what the model sees;
- **Harness Engineering**: organizing tools, memory, skills and the runtime (see [[agentic-harness-engineering]]);
- **Loop Engineering**: iterative execution, reflection and feedback.

All four improve **individual intelligence**. The survey argues that three needs exceed what a single agent can organize: scheduling parallel and interdependent subtasks; keeping specialist roles and verification independent, since an agent judging its own code confuses its belief with evidence; and keeping persistent, recoverable state. The proposed next stage is **System Intelligence**, built through **Graph Engineering**: explicit, dynamic graphs over tasks, agents and runtime state.

## Key points

- **Task organization ("what to do"):** goal-decomposition graphs and workflow optimization. Examples: HuggingGPT, ReWOO, LLMCompiler's dataflow DAG, Plan-over-Graph, and dynamically refined task graphs (TDAG, Flow).
- **Agent coordination ("who works"):** capability modeling, team topology, communication graphs.
- **Runtime state management ("how it operates"):** state recording, fault localization, failure recovery.
- **System evolution:** runtime evidence feeds back into the graph structure.
- **Next frontier, Ontology Engineering:** goal formation and value alignment, shared semantics and world grounding, and measuring system intelligence. Open problems include autonomous ontology construction, self-evolving graph systems and graph-native agent operating systems.
- Long sections catalogue benchmarks and open-source libraries for each tier: model, individual and system intelligence.

## What it does not show

- **No experiments.** It is a position plus taxonomy. "Graph Engineering" is the authors' label for existing work (DAG planners, multi-agent topologies) rather than a validated method.
- **No head-to-head evidence** that explicit graphs beat a single agent with a good harness at matched compute. Wiki sources such as [[diversity-collapse-in-multi-agent-llm-systems-chen-2026]] show multi-agent structure does not help automatically.
- The newsletter one-liner matches the paper. There are no numbers to verify.

## Concepts & entities

- [[agentic-harness-engineering]] · [[context-engineering]] · [[prompt-engineering]] · [[ralph-loop]]: the paradigm ladder.
- [[multi-agent]] · [[agentic-workflow-patterns]] · [[dynamic-workflows]] · [[planning]] · [[knowledge-graph]] · [[graph-memory]]
- [[procedural-graphs-self-evolving-execution-structures-lu-2026]] · [[brain-inspired-graph-multi-agent-systems-hao-2026]] · [[survey-workflow-optimization-llm-agents-yue-2026]]: related graph- and workflow-structured agent work.

## References

- [arXiv:2608.21156](https://arxiv.org/abs/2608.21156)
