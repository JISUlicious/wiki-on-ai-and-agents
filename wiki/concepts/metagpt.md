---
title: MetaGPT
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - hong-metagpt-2023.md
builds-on: "[[llm-agent]]"
status: complete
importance: medium
---

# MetaGPT

MetaGPT is the multi-agent orchestration framework introduced by [[metagpt-hong-2023|Hong et al. (2023)]] that encodes **Standardized Operating Procedures (SOPs)** from human engineering workflows into structured agent-to-agent handoffs. The defining choice: agents don't free-form chat; they pass typed artifacts through an assembly-line pipeline with role-specialized handlers.

For software engineering — the demo domain — the pipeline is:

1. **Product Manager** → produces a PRD (product requirements doc).
2. **Architect** → produces a system architecture doc + API specifications.
3. **Project Manager** → breaks the design into tasks.
4. **Engineer** → writes code for each task.
5. **QA** → writes tests and runs them.

Each role has a system prompt defining its responsibilities and required output schema. The structured handoffs (PRD, architecture doc, task list, code, tests) mean downstream roles can *validate* upstream output, rather than naively trusting free-form text.

## Why structured handoffs help

The MetaGPT paper's core argument: when LLM agents are naively chained in free-form chat, errors compound. Agent A makes a small mistake; agent B can't tell it's wrong; agent C makes a bigger mistake building on it. The structured-handoff approach gives each downstream agent a typed schema to verify upstream output against, catching errors locally before they propagate.

This is a recurring pattern in multi-agent design: the more structure you impose on inter-agent communication, the less you suffer from cascading hallucinations. MetaGPT is the most-cited concrete instantiation; [[autogen|AutoGen]] takes a more general approach (free-form chat with optional structure); production agents like Devin, SWE-agent, and Claude Code mix both depending on the workflow.

## References

- [[metagpt-hong-2023]]
