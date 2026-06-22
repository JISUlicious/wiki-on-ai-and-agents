---
title: Code as Harness
type: concept
created: 2026-05-25
updated: 2026-05-25
sources:
  - code-as-agent-harness-ning-2026.md
  - codeact-wang-2024.md
  - pal-gao-2023.md
  - code-as-policies-liang-2022.md
  - swe-agent-yang-2024.md
  - openhands-wang-2024.md
  - agentcoder-huang-2023.md
  - chatdev-qian-2023.md
  - agentic-harness-engineering-lin-2026.md
  - claude-code-dynamic-workflows-harness-2026.md
  - scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026.md
  - harness-1-state-externalizing-search-agents-jiang-2026.md
status: complete
importance: high
tags:
  - 2026
---

# Code as Harness

**Code as Harness** is the thesis that **executable code is the operational substrate of an agent's harness**, not merely an output the LLM produces. Programs are uniquely *executable* (verifiable by running), *inspectable* (intermediate state as structured traces), and *stateful* (persistent across steps) — making them the only medium that can unify the agent's reasoning, action, environment modeling, and verification under a single abstraction. The concept is formalized by the 2026 survey [[code-as-agent-harness-ning-2026]] (Ning et al., UIUC/Meta/Stanford) but pre-dates it by a long lineage of papers that progressively replaced natural-language and JSON-tool-call abstractions with executable program text.

In the [[agent-three-layer-model]], this is an *axis* spanning L2 — every L2 abstraction (control flow, action surface, context assembly, memory, persistence, harness) can be re-instantiated with code as the medium. The central claim of the 2026 survey is that **the bottleneck of agent autonomy is not the base model's reasoning ability but the reliability of the surrounding harness** — and code is the medium that makes the harness reliable.

## Core mechanism

- **Code for reasoning**: the LLM emits Python (or a DSL) that runs in an interpreter to produce the answer. The interpreter eliminates arithmetic / symbolic errors. Foundational: [[pal-gao-2023|PAL]] (Gao 2023), Program-of-Thoughts (Chen 2022).
- **Code for acting**: actions are emitted as executable function calls or program fragments rather than JSON. Foundational: [[code-as-policies-liang-2022|Code as Policies]] (Liang 2022, embodied/robot); [[codeact-wang-2024|CodeAct]] (Wang 2024, generalist agents).
- **Code for environment**: the world is represented as code state (filesystem, repository, structured trace) rather than free-form text. The agent observes code execution and acts on the same substrate.
- **Verifiable by running**: tests, type checks, linting, and runtime exceptions provide *automatic, deterministic* reward signals — eliminating the reward-model layer for any task that can be verified by code.
- **Stateful across steps**: program state (variables, files, REPL bindings) persists deterministically, replacing fuzzy "memory" with a queryable substrate.

## The harness elements

The survey distinguishes three coupled elements:

1. **Model-internal capabilities** (L1 in the [[agent-three-layer-model]]): reasoning, ICL, alignment.
2. **System-provided harness infrastructure**: planning, memory management, tool use, plan-execute-verify loop, optimization.
3. **Agent-initiated code artifacts** (the novel category): tests, temporary tools, DSL programs, skills, intermediate program states — produced *by* the agent *in response to* a task. The survey's contribution is locating real autonomy gains in this third category.

## Five practical embodiments

| Where | Canonical paper | What it embodies |
|---|---|---|
| Math / scientific reasoning | [[pal-gao-2023|PAL]] | Code-for-reasoning: Python interpreter as offload target for arithmetic / symbolic logic |
| Embodied / robotic | [[code-as-policies-liang-2022|Code as Policies]] | Code-for-acting: LLM emits Python that orchestrates perception + control APIs |
| Generalist agents | [[codeact-wang-2024|CodeAct]] | Replaces JSON tool calls with executable Python; +20% absolute on tool-use evals |
| Software engineering | [[swe-agent-yang-2024|SWE-agent]] + [[openhands-wang-2024|OpenHands]] | Agent-Computer Interface (ACI) abstraction: purpose-built tool surface; SWE-bench 12.5% → 26.7% |
| Multi-agent code collaboration | [[chatdev-qian-2023|ChatDev]] + [[agentcoder-huang-2023|AgentCoder]] + [[metagpt-hong-2023|MetaGPT]] + [[autogen-wu-2023|AutoGen]] | Multiple agents collaborate over a shared program state / test suite |

## Frontier — Agentic Harness Engineering

The survey's §3.5 introduces a 2026 frontier: the harness itself becomes the object of optimization — telemetry-driven mutation, program search, self-synthesis, and scaling laws over the scaffold, with the base model held fixed. This has grown into its own concept; see **[[agentic-harness-engineering]]** for the full treatment and source cluster. The intellectual line runs from the [[godel-machine|Gödel-machine]] / [[darwin-godel-machine]] (which evolve the *agent*) to AHE (which evolves only the *harness*), adjacent to [[agentic-context-engineering]] (prompt-level evolution).

## Frontier — self-authored harness ([[dynamic-workflows]])

A second 2026 instantiation, complementary to harness *mutation*: in [[claude-code|Claude Code]], Claude **writes a fresh harness per task as executable code**. [[dynamic-workflows|Dynamic workflows]] (Shihipar & Bidasaria, [[claude-code-dynamic-workflows-harness-2026|2026]]) have Claude author a JavaScript orchestration script that spawns and coordinates parallel subagents — the harness *is* code the agent generates on the fly, specialized to the task, rather than a fixed scaffold. Where [[agentic-harness-engineering]] *evolves* a standing harness via telemetry, dynamic workflows *generate* a disposable one per task. Both are the [[code-as-harness]] thesis turned on the harness itself; the explicit motivation is that a single context window suffers agentic laziness, self-preferential bias, and goal drift — failures that code-orchestrated, independently-verified subagents avoid.

## Relation to other concepts

- [[dynamic-workflows]] — Claude writing its own multi-agent harness as code per task (the self-authored endpoint of this axis).
- [[agent-three-layer-model]] — Code-as-harness is an orthogonal axis across L2: every L2 abstraction can be reinstantiated in code.
- [[function-calling]] / [[tool-use]] — Code-as-harness *generalizes* function calling: instead of one JSON call, the agent emits arbitrary Python.
- [[mcp]] — MCP standardizes tool exposure but is agnostic about what the agent emits. Code-as-harness says the agent should emit code, not JSON, for arbitrary actions.
- [[react]] — The reasoning-acting loop; code-as-harness changes *what's in the Act slot* from a tool name to executable code.
- [[self-improving-agent]] — Once the harness is code, it can be mutated by the agent. [[agentic-harness-engineering]] is the canonical 2026 instantiation.
- [[reward-modeling]] — Where verifiable by code (tests pass / answer correct), the reward model disappears entirely (see [[deepseek-r1]], [[search-r1]], [[swe-gym]] outcome rewards).

## References

- [[code-as-agent-harness-ning-2026]] — the 2026 survey that names and frames the concept (66 pages, 500+ references).
- [[pal-gao-2023]] — code-for-reasoning origin.
- [[code-as-policies-liang-2022]] — code-for-acting origin (embodied).
- [[codeact-wang-2024]] — generalist-agent embodiment.
- [[swe-agent-yang-2024]] — Agent-Computer Interface for SE.
- [[openhands-wang-2024]] — open-source reference platform.
- [[agentcoder-huang-2023]] — most-cited paper in the survey body (19 citations); deterministic test-executor as harness component.
- [[chatdev-qian-2023]] — multi-agent code collaboration.
- [[agentic-harness-engineering-lin-2026]] — 2026 frontier: harness as object of optimization.
- [[scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026]] — Effective Feedback Compute: harness scaling is governed by feedback *quality*, not raw token/tool budget.
- [[harness-1-state-externalizing-search-agents-jiang-2026]] — Harness-1: offloads recoverable state to an environment-side working memory so RL optimizes only genuine search decisions.
- [[harness-updating-is-not-harness-benefit-lin-2026]] — self-evolving harnesses: updating quality is flat across model tiers; the benefit of a better harness is non-monotonic.
- [[leap-formal-mathematics-agentic-kung-2026]] — a well-built Lean-verifier harness lets a general LLM match dedicated frontier math models (all 12 Putnam-2025).

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]] for full grouping.

- [[autonomous-long-horizon-ml-research-engineering-chen-2026]] — Toward Autonomous Long Horizon Engineering for ML Research
- [[coding-agents-effective-long-context-processors-cao-2026]] — Coding Agents are Effective Long Context Processors
- [[dive-into-claude-code-design-space-liu-2026]] — Dive into Claude Code The Design Space of
- [[effective-strategies-asynchronous-swe-agents-geng-2026]] — Effective Strategies for Asynchronous Software Engineering Agents
- [[learning-agent-compatible-context-management-yi-2026]] — Learning Agent Compatible Context Management for Long Horizon
- [[llm-diagnosis-of-integration-test-failures-ziftci-2026]] — LLM Based Automated Diagnosis Of Integration Test Failures
- [[opendev-terminal-coding-agent-bui-2026]] — Building Effective AI Coding Agents for the Terminal
- [[structuredagent-andor-tree-planning-lobo-2026]] — StructuredAgent Planning with AND/OR Trees for Long Horizon
- [[fundamental-components-of-harness]] — Fundamental Components of an LLM Agent Harness
- [[spatialclaw-cho-2026]] — SpatialClaw: code-as-action spatial reasoning over a persistent kernel (NVIDIA)
