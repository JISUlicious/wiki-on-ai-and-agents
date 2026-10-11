---
title: "Optimize agents (Google ADK documentation)"
type: source
created: 2026-10-08
updated: 2026-10-08
sources:
  - adk-optimize-docs-2026.md
year: 2026
venue: "adk.dev documentation"
url: https://adk.dev/optimize/
tags:
  - 2026
  - docs
status: complete
importance: medium
---

# Optimize agents (Google ADK documentation)

**Source**: `sources/adk-optimize-docs-2026.md` ([adk.dev/optimize](https://adk.dev/optimize/), fetched 2026-10-08)
**Publisher**: [[google|Google]], for the Agent Development Kit ([[google-adk]])

## Summary

ADK has "an extendable framework for automated agent optimization based on evaluation results." The page header says "Supported in ADK Python v1.24.0".

There are two parts:
- a **Sampler**, which evaluates candidate agents against eval sets. The built-in `LocalEvalSampler` uses train and validation eval sets.
- an **Agent Optimizer**, which reads the evaluation results and proposes improved agents.

The CLI entry point is `adk optimize [OPTIONS] AGENT_MODULE_FILE_PATH`, with an optional `--optimizer_config_file_path`. The docs' worked example adds a rule to a hello-world agent that the optimizer is expected to discover from the eval cases.

## The optimizers

| Optimizer | What it optimizes | Notes |
|---|---|---|
| **GEPARootAgentPromptOptimizer** | The **root agent's instructions**, using [[gepa\|GEPA]] | "does not improve any sub-agents, agent tools, skills, or any other aspect of the root agent." Experimental. Defaults: `max_metric_calls` 100, `reflection_minibatch_size` 3, a 10K-token thinking budget for the optimizer model. `run_dir` allows warm starts. |
| **GEPARootAgentOptimizer** | The root agent's instructions **and the instructions of skills supplied through a `SkillToolset`** | "an extension of" the prompt optimizer. Does not improve sub-agents or tools. Experimental. Optimizer thinking level HIGH. |
| **SimplePromptOptimizer** | The root system instructions, in a sequential execute → evaluate → critique → rewrite loop | "Unlike the GEPA-based optimizers that maintain a diverse Pareto frontier", it refines a single prompt and returns the best-scoring variant. |

## Version history (verified 2026-10-08)

Verified by inspecting the PyPI wheels of `google-adk` (classes and CLI commands present in each version) and PyPI upload dates. The analysis-docs session gathered the version list; this wiki spot-checked 1.26.0 vs 1.27.0 and 2.2.0 vs 2.3.0.

| Version (PyPI date) | What appears |
|---|---|
| **1.24.0** (2026-02-05) | Optimization interfaces (`agent_optimizer`, `sampler`, data types). This is what the docs header "Supported in ADK Python v1.24.0" refers to. |
| 1.25.0 | `SimplePromptOptimizer` (reported by analysis-docs; not spot-checked) |
| **1.27.0** (2026-03-12) | **`adk optimize` CLI**, **GEPARootAgentPromptOptimizer** and `LocalEvalSampler`. None of the three are in 1.26.0. Matches the v1.27.0 release notes. |
| **2.3.0** (2026-06-18) | **GEPARootAgentOptimizer** (root plus skill instructions) first appears. It is absent in 2.2.0 and absent from the whole 1.x line (latest 1.39.1). |
| 2.11.0 (2026-10-02) | The `adk optimize` CLI still uses GEPARootAgentPromptOptimizer (root instructions only). GEPARootAgentOptimizer is available **only through the Python API**. |

**Other details, from the ADK source (reported by analysis-docs):**
- `gepa>=0.1` is installed with the `google-adk[eval]` extra.
- `LocalEvalSampler` scores each example as a binary pass/fail (1.0/0.0).
- If `validation_eval_set` is omitted, it validates on the training set and warns about train/validation overlap.

> [!success] Resolved: versions
> **Resolved 2026-10-08:** an earlier version of this page recorded "1.27.0+" and "GEPARootAgentOptimizer in 2.11.0" as unverified. 1.27.0 is confirmed for the CLI. The **2.11.0 claim was wrong**: GEPARootAgentOptimizer was introduced in **2.3.0**, and 2.11.0 is only a version in which it exists.

## Why it matters for the wiki

GEPA moving from a research optimizer into a mainstream agent framework is an adoption marker for automated [[prompt-optimization]] of agents.

The progression from root-instruction optimization to instructions plus **skills** parallels the wiki's [[skill-optimization]] thread. Sub-agents and tools are explicitly out of scope, which is the boundary that harness-evolution research ([[agentic-harness-engineering]]) tries to cross.

## Concepts & entities

[[google-adk]] · [[gepa]] · [[prompt-optimization]] · [[skill-optimization]] · [[agent-skills]] · [[agent-evaluation]] · [[google]]
