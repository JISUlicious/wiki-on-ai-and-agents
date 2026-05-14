---
title: Standardized Operating Procedures (SOPs) in LLM Agents
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - hong-metagpt-2023.md
status: draft
importance: low
---

# Standardized Operating Procedures (SOPs) in LLM Agents

In the LLM-agent literature, SOPs refer to **predefined sequences of role-specialized handoffs with typed intermediate artifacts**, borrowed from human-organizational workflows. Introduced as a multi-agent design pattern in [[metagpt-hong-2023|MetaGPT]] (Hong et al. 2023).

The idea is in tension with the dominant "let the LLM figure out what to do next" approach ([[react|ReAct]], single-agent loops, AutoGen-style free-form chat). SOPs make the workflow *explicit* and the inter-agent interfaces *typed* — which dramatically reduces cascading-hallucination errors at the cost of flexibility.

When SOPs help:

- The workflow is well-understood (e.g., software engineering, customer support).
- Multiple specialized roles add value (a Product Manager *should* think differently from an Engineer).
- Intermediate artifacts have natural typed structure (a PRD is a doc; a task list is a list; code is code).

When SOPs don't help:

- Open-ended exploration.
- Workflows that don't fit any predefined template.
- Tasks where a single agent's flexibility outperforms the overhead of role handoffs.

The practical 2024–2025 picture is hybrid: many production agentic systems use SOP-style structure for the *outer* workflow (a deterministic pipeline of stages) and free-form ReAct-style behavior within each stage.

## References

- [[metagpt-hong-2023]]
