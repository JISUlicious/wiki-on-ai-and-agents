---
title: Agentic Workflow Patterns (workflows vs agents)
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - building-effective-agents-anthropic-2024.md
  - effective-context-engineering-anthropic-2025.md
status: complete
importance: high
---

# Agentic Workflow Patterns

The vocabulary [[anthropic|Anthropic]] set out in [[building-effective-agents-anthropic-2024|Building effective agents]] (2024-12-19). All of these are **agentic systems**, split two ways:

- **Workflows**: "LLMs and tools are orchestrated through predefined code paths."
- **Agents**: "LLMs dynamically direct their own processes and tool usage."

The governing advice: start with the simplest solution, often a single well-prompted call with retrieval. Add complexity only when it measurably helps.

## The building block and five workflows

| Pattern | What it is | Use when |
|---|---|---|
| **Augmented LLM** | One call with retrieval, tools and memory, e.g. via [[mcp\|MCP]] | Always the base unit |
| **[[prompt-chaining\|Prompt chaining]]** | Fixed sequence of calls with optional gates | The task splits cleanly into fixed subtasks |
| **Routing** | Classify, then dispatch to a specialized prompt or model | Distinct input categories; easy vs hard queries |
| **Parallelization** | *Sectioning* (independent subtasks) or *voting* (same task several times) | Speed, or confidence from multiple attempts; guardrails as a separate call |
| **Orchestrator-workers** | A central LLM picks subtasks at run time, delegates and merges | The subtasks can't be predicted, e.g. multi-file code changes |
| **Evaluator-optimizer** | Generate → critique → revise loop | Clear evaluation criteria, where feedback measurably improves output |

## Agents

"Typically just LLMs using tools based on environmental feedback in a loop." By September 2025, Anthropic had narrowed this to "**LLMs autonomously using tools in a loop**" ([[effective-context-engineering-anthropic-2025]]).

## How the patterns map to later wiki concepts

- **Orchestrator-workers** → [[subagents]], [[multi-agent]], and Claude Code's [[dynamic-workflows]]. There, the orchestrator is code that Claude writes itself, so the workflow/agent line blurs.
- **Evaluator-optimizer** → [[verifier]] loops, [[reflexion|Reflexion]], and [[llm-as-a-judge]]. It is the same generator/critic split as RL's [[actor-critic]], by analogy only.
- **Voting** → [[self-consistency]] and best-of-N ([[test-time-compute]]).
- **The augmented LLM's tools** → [[agent-computer-interface]]. The post's appendix makes ACI design a first-class concern.

## Classical precedent

Workflows vs agents echoes the classical split between agents whose control is fixed by their designer and agents that choose actions from percepts and goals ([[intelligent-agent]]).

## References

- [[building-effective-agents-anthropic-2024]]
- [[effective-context-engineering-anthropic-2025]]
