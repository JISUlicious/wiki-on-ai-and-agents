---
title: Prompt Chaining
type: concept
created: 2026-10-06
updated: 2026-10-07
sources:
  - ai-chains-wu-2021.md
  - building-effective-agents-anthropic-2024.md
  - langchain-readme.md
status: complete
importance: high
---

# Prompt Chaining

Splitting a task into a **fixed sequence of LLM calls**, where each call's output becomes the next call's input. Code can check the intermediate results, acting as *gates*, before the chain continues. It is the simplest multi-call pattern and the ancestor of most LLM orchestration.

## Lineage

| When | Where | What it added |
|---|---|---|
| Oct 2021 | [[ai-chains-wu-2021\|AI Chains]] (Wu, Terry & Cai; CHI 2022) | Names **Chaining**; defines primitive operations (classify, ideate, split, compose, rewrite…). Its user study finds gains in quality, transparency and control, and users "unit-test" individual steps. |
| Oct 2022 | [[langchain\|LangChain]] (repo created 2022-10-17) | Makes "chains" a developer abstraction; later adds agents, LangGraph and Deep Agents. |
| Dec 2024 | [[building-effective-agents-anthropic-2024\|Building effective agents]] (Anthropic) | Lists prompt chaining as the first of five **workflows**: use it when the task "can be easily and cleanly decomposed into fixed subtasks", trading latency for accuracy. Adds programmatic gates. |
| 2026 | [[dynamic-workflows]] | The model writes the orchestration, including chains, as code at run time. |

## Chaining vs agents

A chain's **path is fixed in advance** by a human. An agent ([[react|ReAct]] and its successors) chooses its next step at run time from environmental feedback. Anthropic's terms:
- chaining, routing and parallelization are *workflows*;
- [[agentic-workflow-patterns|orchestrator-workers]] is the first workflow where the subtasks are not predefined.

## When it helps (per the sources)

- Each step becomes an easier task, which improves accuracy at the cost of latency.
- Intermediate outputs can be inspected and edited, which improves transparency and debuggability.
- Generic steps such as "outline, then check, then write" can be reused.

## Related

[[agentic-workflow-patterns]] · [[langchain]] · [[llm-agent]] · [[dynamic-workflows]] · [[in-context-learning]] · [[chain-of-thought-prompting]], which decomposes *within* one call rather than across calls.

## References

- [[ai-chains-wu-2021]]
- [[building-effective-agents-anthropic-2024]]
- [[langchain-readme-2026]]

## Lineage sources

- [[langchain-history-2024-2025]] — LCEL chains as DAGs, and why agents needed cycles
