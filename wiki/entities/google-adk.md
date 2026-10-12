---
title: Google Agent Development Kit (ADK)
type: entity
created: 2026-10-08
updated: 2026-10-12
sources:
  - adk-optimize-docs-2026.md
status: draft
importance: medium
tags:
  - product
---

# Google Agent Development Kit (ADK)

[[google|Google]]'s open-source framework for building, evaluating and deploying agents. Docs are at [adk.dev](https://adk.dev/); there are Python and JavaScript versions.

## Automated optimization

Documented at [[adk-optimize-docs-2026]] ("Supported in ADK Python v1.24.0"):
- **`adk optimize`** runs a Sampler (eval-based scoring) together with an Agent Optimizer.
- **GEPARootAgentPromptOptimizer** uses [[gepa|GEPA]] to optimize the **root agent's instructions** only.
- **GEPARootAgentOptimizer** optimizes the root instructions **plus skill instructions** supplied through a `SkillToolset`. Sub-agents and tools are not optimized.
- **SimplePromptOptimizer** is a sequential critique-and-rewrite loop.
- All the GEPA optimizers are marked **experimental**.

**Versions** (verified against PyPI wheels; see [[adk-optimize-docs-2026]]):
- optimization interfaces arrived in **1.24.0** (2026-02);
- the `adk optimize` CLI and GEPARootAgentPromptOptimizer in **1.27.0** (2026-03-12);
- GEPARootAgentOptimizer in **2.3.0** (2026-06-18).

As of 2.11.0, the CLI still optimizes root instructions only; skill co-optimization is available through the Python API.

## System One / decision models

**No official support as of 2.11.0** ([[adk-system-one-integration-status-2026]]):
- **PR #7235 is open and unmerged.** It is a community TypeSafe (Jev) integration proposed as `google.adk.integrations.typesafe`: a classifier, a single-question tool, and callbacks for model routing and tool gating. It is built that way because decision models don't fit the `BaseLlm` contract.
- **ADK's LiteLLM bridge can't reach decision models.** Its prefix allowlist excludes `typesafe/`, and it calls completion APIs only.
- **`tools/model_consult/` doesn't apply.** It resolves `BaseLlm` models.

## Related

[[gepa]] · [[prompt-optimization]] · [[skill-optimization]] · [[agent-skills]] · [[agent-evaluation]] · [[langchain]] · [[dspy]]

## References

- [[adk-optimize-docs-2026]]
