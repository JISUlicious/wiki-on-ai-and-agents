---
title: "LLM Powered Autonomous Agents"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - llm-powered-autonomous-agents-weng-2023.md
authors:
  - "[[lilian-weng]]"
year: 2023
venue: "Lil'Log (lilianweng.github.io)"
url: https://lilianweng.github.io/posts/2023-06-23-agent/
tags:
  - 2023
  - blog
status: complete
importance: high
---

# LLM Powered Autonomous Agents

**Source**: `sources/llm-powered-autonomous-agents-weng-2023.md` ([lilianweng.github.io/posts/2023-06-23-agent](https://lilianweng.github.io/posts/2023-06-23-agent/))
**Author**: [[lilian-weng]] (then at [[openai]])
**Date**: 2023-06-23

## Summary

The most-cited early survey of LLM agents, written three months after [[autogpt|AutoGPT]] and [[babyagi|BabyAGI]] went viral. Its one-line framing is that the LLM "functions as the agent's brain", complemented by three components. That **planning / memory / tool use** breakdown became the default way to describe an agent.

## The three components

**1. Planning**
- *Task decomposition*:
  - [[chain-of-thought-prompting|CoT]], and [[tree-of-thoughts|Tree of Thoughts]] (BFS or DFS over thoughts).
  - Simple prompting ("Steps for XYZ").
  - Human input.
  - LLM+P, which hands long-horizon planning to a classical PDDL planner.
- *Self-reflection*: [[react|ReAct]]'s Thought → Action → Observation loop, [[reflexion|Reflexion]], Chain of Hindsight and Algorithm Distillation.

**2. Memory.** Weng maps the categories of human memory onto agent parts:
- **sensory memory** → embedding representations of raw inputs;
- **short-term memory** → [[in-context-learning|in-context learning]], "restricted by the finite context window";
- **long-term memory** → an external vector store, reached through fast maximum inner product search (MIPS). The post compares LSH, ANNOY, HNSW, FAISS and ScaNN.

**3. Tool use.**
- MRKL: an LLM routes queries to neural or symbolic expert modules.
- TALM and [[toolformer|Toolformer]] fine-tune the model to call APIs.
- [[chatgpt-plugins|ChatGPT plugins]] and OpenAI [[function-calling|function calling]] are "good examples… working in practice". Plugins expose tools written by third parties; function calls are defined by the developer.
- HuggingGPT, with four stages: task planning, model selection, task execution and response generation.
- API-Bank, a benchmark for tool-using LLMs.

## Case studies

- ChemCrow and the Boiko et al. scientific-discovery agents, including the risk of synthesizing dangerous chemicals.
- [[generative-agents|Generative Agents]]: memory stream, retrieval by recency, importance and relevance, then reflection and planning.
- [[autogpt|AutoGPT]] (its full system prompt is quoted) and GPT-Engineer.

## Three challenges named in 2023

1. **Finite context length.** It limits history, instructions and API context. Self-reflection "would benefit a lot from long or infinite context windows."
2. **Long-term planning and task decomposition.** LLMs struggle to adjust plans after unexpected errors.
3. **Reliability of the natural-language interface.** Formatting errors and refusals mean "much of the agent demo code focuses on parsing model output."

All three map onto later wiki threads:
1. Context length → [[long-context-llm]] and [[context-engineering]].
2. Planning → [[planning]] and [[verifier]]-gated loops.
3. Interface reliability → structured [[function-calling]] and typed outputs.

## Why it matters for the wiki

This post sets the canonical 2023 component model. The wiki's [[agent-three-layer-model]] and the 2023 memory papers ([[cognitive-architectures-for-language-agents|CoALA]], [[memgpt|MemGPT]]) refine it. User-designated trusted source (Lil'Log).

## Concepts & entities

[[llm-agent]] · [[planning]] · [[memory-management]] · [[long-term-memory]] · [[tool-use]] · [[chain-of-thought-prompting]] · [[tree-of-thoughts]] · [[react]] · [[reflexion]] · [[generative-agents]] · [[toolformer]] · [[autogpt]] · [[babyagi]] · [[chatgpt-plugins]] · [[function-calling]] · [[lilian-weng]]
