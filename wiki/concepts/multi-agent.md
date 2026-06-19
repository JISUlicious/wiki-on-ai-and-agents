---
title: Multi-Agent Systems (LLM)
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - reaching-agreement-among-reasoning-llm-agents-ruan-2025.md
  - single-agent-vs-multi-agent-tran-2026.md
  - diversity-collapse-in-multi-agent-llm-systems-chen-2026.md
  - drop-the-hierarchy-and-roles-dochkina-2026.md
  - reliability-limits-llm-multi-agent-planning-ao-2026.md
  - claude-code-dynamic-workflows-harness-2026.md
  - scaling-behavior-single-llm-multi-agent-systems-li-2026.md
status: draft
importance: high
tags:
  - 2026
---

# Multi-Agent Systems (LLM)

**Multi-agent systems (MAS)** coordinate multiple [[llm-agent|LLM agents]] — each with its own role, context, or tool access — toward a shared task, communicating through messages, shared memory, or a blackboard. The premise is that decomposition + specialization + debate beats a single monolithic agent. Canonical patterns include [[metagpt|role-play pipelines]], [[autogen|conversational orchestration]], debate/consensus, and planner–executor hierarchies. This page is the general MAS concept; for the memory substrate shared across agents see [[multi-agent-memory]].

## Coordination structures

- **Designed hierarchies / roles** — a fixed org chart (manager → workers, or SOP-driven role play as in [[metagpt]]). [[from-skills-to-talent-organising-heterogeneous-agents-yu-2026|"From Skills to Talent"]] pushes this to a company metaphor with heterogeneous agents.
- **Self-organizing** — [[drop-the-hierarchy-and-roles-dochkina-2026|Dochkina et al. 2026]] report that *self-organizing* agents can outperform hand-designed structures, questioning the value of rigid role assignment.
- **Debate / consensus** — agents argue toward agreement ([[reaching-agreement-among-reasoning-llm-agents-ruan-2025]]); [[latent-agents-internalized-multi-agent-debate-yi-2026|Latent Agents]] internalizes the debate into a single model's forward pass via post-training.
- **Recursive / graph** — agents spawn sub-agents recursively ([[recursive-multi-agent-systems-yang-2026]]) or pass messages over a graph topology ([[brain-inspired-graph-multi-agent-systems-hao-2026]], [[hyperagents-zhang-2026]]).
- **Deterministic code orchestration** — instead of agents conversing, a *program* spawns and coordinates them. [[dynamic-workflows|Dynamic workflows]] in [[claude-code|Claude Code]] ([[claude-code-dynamic-workflows-harness-2026|Shihipar & Bidasaria 2026]]) have Claude write a JavaScript script that fans out to parallel subagents with patterns like fan-out-and-synthesize, tournament, and adversarial verification. Control flow is deterministic JS; only the leaf steps are LLM calls — the opposite end of the spectrum from free-form agent debate, and a direct response to the diversity-collapse / reliability concerns below.

## When does MAS actually help?

A recurring 2026 theme is skepticism — measuring whether multi-agent coordination beats a single agent given equal compute:

- [[single-agent-vs-multi-agent-tran-2026|Tran et al. 2026]] find a **single agent outperforms multi-agent systems** on multi-hop reasoning *under equal thinking-token budgets* — the apparent MAS advantage is often just more total compute.
- [[diversity-collapse-in-multi-agent-llm-systems-chen-2026|Diversity Collapse]] shows agents drawn from the same base model converge to near-identical outputs, eroding the diversity that debate/ensembling assumes.
- [[reliability-limits-llm-multi-agent-planning-ao-2026|Reliability Limits]] characterizes error-compounding in multi-agent planning chains.
- [[scaling-behavior-single-llm-multi-agent-systems-li-2026|SIMAS]] (Li et al. 2026) shows performance is **non-monotonic** in agent count — diminishing returns then degradation from coordination overhead; the optimal count is task-dependent, so "design the interaction, don't just stack agents."
- [[theory-of-mind-internal-beliefs-multi-agent-kostka-2026|ToM evaluation]] probes whether agents maintain accurate internal beliefs about each other — a prerequisite for genuine coordination.

These results push the field toward *equal-compute* baselines and away from uncritical "more agents = better" framing.

## Related

- [[llm-agent]] — the single-agent building block.
- [[multi-agent-memory]] — shared/partitioned memory across agents.
- [[metagpt]], [[autogen]] — foundational MAS frameworks.
- [[mass-rag-multi-agent-synthesis-rag-xiao-2026]] — multi-agent retrieval-augmented synthesis.
- [[autonomous-evolution-eda-tools-multi-agent-abc-yu-2026]] — MAS applied to EDA tool evolution.

## References

- [[single-agent-vs-multi-agent-tran-2026]]
- [[diversity-collapse-in-multi-agent-llm-systems-chen-2026]]
- [[drop-the-hierarchy-and-roles-dochkina-2026]]
- [[reaching-agreement-among-reasoning-llm-agents-ruan-2025]]
- [[reliability-limits-llm-multi-agent-planning-ao-2026]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]] for full grouping.

- [[multi-user-llm-agents-yang-2026]] — Multi User Large Language Model Agents
- [[survey-workflow-optimization-llm-agents-yue-2026]] — From Static Templates to Dynamic Runtime Graphs A
