---
title: "IBM: AI Agent Evaluation (tutorial)"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
year: 2026
venue: "IBM Think tutorials"
authors:
  - Shalini Harkar
tags:
  - 2026
status: complete
importance: low
---

# IBM: AI Agent Evaluation (tutorial)

[ibm.com/think/tutorials/ai-agent-evaluation](https://www.ibm.com/think/tutorials/ai-agent-evaluation) — an educational tutorial under IBM Think's AI-governance section. Its value is the **taxonomy**, not the implementation.

## Seven reasons to evaluate agents

Functional verification · design optimization · **robustness** (edge cases, adversarial inputs, fault tolerance) · performance and resource metrics · user interaction quality · goal completion analysis · **ethical and safety considerations**.

Robustness and ethics/safety are dimensions the developer-tool documentation ([[langsmith-evaluation-docs]], [[mlflow-genai-evaluation-docs]]) largely treats as separate concerns rather than part of agent evaluation proper.

## Three formal metric classes

| Class | Contains |
|---|---|
| **Accuracy** | correctness, helpfulness, coherence |
| **Response time** | throughput, average latency, timeout delay |
| **Cost** | token usage, compute time, API call count, memory consumption |

The definitions are clean and worth keeping:
- **Correctness** — factually accurate and logically true given the input.
- **Helpfulness** — *"how useful or actionable the response is for the user's intent. Even if a response is factually correct, it might not be helpful if it does not address a solution or next steps."*
- **Coherence** — logical and narrative flow; *"important in multiturn interactions... whether the agent 'makes sense' from start to finish."*

## Method, and a caveat

Pure LLM-as-a-judge: a single LangChain `PromptTemplate` scoring all three criteria on a **1-5 scale**. No trajectory evaluation, no tool-call scoring, no dataset machinery, no online/offline distinction.

> [!warning] The agent grades itself
> The evaluation prompt is passed back into `agent.invoke(...)` — the same agent under test produces the scores. Self-evaluation without an independent judge is a methodological flaw worth noting; contrast with [[agent-as-a-judge-zhuge-2024]]'s explicit calibration against human consensus.

The stack is watsonx.ai with `granite-3-8b-instruct`; the conclusion recommends LangGraph and IBM watsonx.governance.

## Concepts

- [[agent-evaluation]] · [[llm-as-a-judge]]

## References

- [IBM: AI agent evaluation](https://www.ibm.com/think/tutorials/ai-agent-evaluation)
