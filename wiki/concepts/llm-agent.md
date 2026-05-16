---
title: LLM Agent
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - react-yao-2022.md
status: complete
importance: high
---

# LLM Agent

An **LLM agent** is a system in which a large language model is placed inside a perception-action loop, given a goal, and allowed to take actions against some environment until the goal is achieved (or it gives up). The defining characteristics:

- **Goal-directed**: the system pursues an objective specified by the user.
- **Iterative**: it operates over many LLM calls / actions, not a single prompt-response.
- **Tool-augmented**: it can take actions via [[tool-use|external tools]] (search, code, APIs, file system, browser, robot).
- **Self-directed**: it decides what to do next based on its own reasoning, not a hardcoded plan.

## Canonical loop

The dominant pattern is **[[react|ReAct]]**-style:

```
while not done:
    thought = LLM.generate(history + scratchpad)
    action  = parse_action(thought)
    obs     = environment.execute(action)
    history.append(thought, action, obs)
```

Many variants exist:

- **Planner + executor** (BabyAGI, AutoGPT): an explicit planning LLM produces a task list, which an executor LLM works through.
- **Multi-agent** (CAMEL, AutoGen, MetaGPT): multiple LLM instances with different roles converse.
- **Tree search over actions** (Tree-of-Thoughts, Yao et al. 2023; LATS): explicit search over the action space.
- **Self-reflection** ([[reflexion|Reflexion]]): the agent evaluates and revises its own past trajectories.

## Memory

Beyond the immediate context window, agents manage memory across an explicit lifecycle — see [[memory-management]] for the full vocabulary (write/store/retrieve/compact/consolidate/evict/curate) and canonical works. Common substrates:

- **Scratchpad**: in-context working memory, lost between sessions.
- **Vector store** (semantic memory): retrieve relevant past observations via embedding similarity.
- **Structured memory** (key-value stores, databases): explicit reads/writes — see [[memgpt]], [[mem0]].
- **Graph memory**: relational stores with PPR-style retrieval — see [[hipporag-2]], [[a-mem]].
- **Procedural memory**: skill libraries the agent grows over time — see [[voyager]].
- **Multi-typed memory**: Core / Episodic / Semantic / Procedural / Resource / Vault — see [[mirix]].
- **Parametric/online**: weights updated at test time as memory — see [[titans]].
- **KV-cache-internal**: episodic structure inside the context cache — see [[em-llm]].
- **Long-context**: 1M+ token windows reduce the need for external memory for short-to-medium tasks (but the lifecycle questions remain).

## Production state (as of 2026)

Mainstream production agent stacks combine:

- A strong instruction-tuned model (Claude, GPT-4, Gemini).
- Structured tool-use APIs (OpenAI function calling, Anthropic tool use, **MCP** servers).
- A scaffolding/orchestration framework (LangGraph, Anthropic's Computer Use, OpenAI's Assistants API).
- Domain-specific tools and memory.

Agentic capability is one of the most active commercial frontiers — coding agents (Cursor, Claude Code, Devin), browser/computer-use agents, and customer-service/automation agents are the visible examples.

## Open problems

- **Reliability over long horizons**: agents drift, get stuck, or make catastrophic errors over long task sequences.
- **Cost**: agentic workflows can consume thousands of LLM tokens per user request.
- **Evaluation**: hard to benchmark; "did the agent succeed" is task-specific and often subjective.
- **Safety**: an autonomous LLM with arbitrary actuation is a non-trivial security surface (cf. prompt injection, sandbox escape).

See [[agentic-ai-and-the-next-intelligence-explosion]] for a recent essay-length argument about where this is going.

## References

- [[react-yao-2022]]
