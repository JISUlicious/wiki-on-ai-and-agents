---
title: "Building effective agents"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - building-effective-agents-anthropic-2024.md
authors:
  - Erik Schluntz
  - Barry Zhang
introduces:
  - "[[agentic-workflow-patterns]]"
  - "[[prompt-chaining]]"
year: 2024
venue: Anthropic Engineering blog
url: https://www.anthropic.com/engineering/building-effective-agents
tags:
  - 2024
  - blog
status: complete
importance: high
---

# Building effective agents

**Source**: `sources/building-effective-agents-anthropic-2024.md` ([anthropic.com/engineering/building-effective-agents](https://www.anthropic.com/engineering/building-effective-agents); originally at `/research/`)
**Authors**: Erik S. and Barry Zhang ([[anthropic]]). The page credits "Erik S."; the full surname is from general knowledge.
**Date**: published 2024-12-19. The live page now carries a note that much of the tooling it describes has changed since then.

## Summary

Anthropic's canonical framing of agent design, drawn from work with "dozens of teams". The central claim is that the most successful systems use **simple, composable patterns rather than complex frameworks**, and that complexity should be added only when it measurably improves results.

The post's lasting contribution is a vocabulary. It groups everything under **agentic systems** and splits them in two:
- **Workflows**: "LLMs and tools are orchestrated through predefined code paths."
- **Agents**: "LLMs dynamically direct their own processes and tool usage."

## The building block and five workflows

- **The augmented LLM**: a model with retrieval, tools and memory. [[mcp|MCP]], released the previous month, is named as one way to supply them.
- **[[prompt-chaining|Prompt chaining]]**: a fixed sequence of calls, each consuming the previous output, with optional programmatic *gates* between steps. It trades latency for accuracy by making each call easier.
- **Routing**: classify the input, then send it to a specialized prompt, tool or model, such as easy questions to a small model and hard ones to a large one.
- **Parallelization**: either *sectioning* (independent subtasks in parallel) or *voting* (the same task several times). Guardrails run as a separate call are an example.
- **Orchestrator-workers**: a central LLM decides the subtasks at run time, delegates them and merges the results. The difference from parallelization is that the subtasks are not predefined.
- **Evaluator-optimizer**: one call generates, another critiques, in a loop. It fits when evaluation criteria are clear and human-style feedback would demonstrably help.

## Agents

Agents are "typically just LLMs using tools based on environmental feedback in a loop." They need "ground truth" from the environment at every step, such as tool results or code execution. They suit open-ended problems where the number of steps cannot be predicted. The costs are higher spend and compounding errors, so the post recommends sandboxed testing and guardrails. Anthropic's examples are its SWE-bench coding agent and its computer-use reference implementation.

**Three principles:** keep the design simple; make planning steps transparent; and carefully craft the **agent-computer interface** ([[agent-computer-interface|ACI]]) through tool documentation and testing.

## Appendix highlights

- **Where agents pay off:** customer support and coding, because both combine conversation with action, have clear success criteria and allow feedback loops. Code is verifiable by automated tests, though human review is still needed.
- **"Prompt engineering your tools":** give tools as much prompt-engineering attention as the main prompt.
  - Choose formats the model writes easily. A diff needs the hunk's line count before the code is written, and code inside JSON must be escaped.
  - "Poka-yoke" the arguments so mistakes are hard to make.
  - On SWE-bench, Anthropic "spent more time optimizing our tools than the overall prompt." Requiring absolute file paths eliminated a class of errors.

## Why it matters for the wiki

This is the origin of the wiki's **workflow vs agent** distinction and its pattern names; see [[agentic-workflow-patterns]]. Later lineage:
- The [[effective-context-engineering-anthropic-2025|context-engineering post]] (Sep 2025) narrows the definition of an agent to "LLMs autonomously using tools in a loop".
- [[dynamic-workflows]] (2026) collapses the split: the model writes the orchestration code that this post assumed a human would write.
- Its evaluator-optimizer pattern is the workflow form of the [[verifier]] loop.

## Concepts & entities

[[agentic-workflow-patterns]] · [[prompt-chaining]] · [[llm-agent]] · [[agent-computer-interface]] · [[tool-use]] · [[mcp]] · [[multi-agent]] · [[verifier]] · [[anthropic]] · [[swe-bench-verified]] · [[computer-use-agent]]
