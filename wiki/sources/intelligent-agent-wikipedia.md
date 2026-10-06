---
title: "Intelligent agent (Wikipedia)"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - intelligent-agent-wikipedia.md
introduces:
  - "[[intelligent-agent]]"
year: 2026
venue: Wikipedia (fetched 2026-10-06)
url: https://en.wikipedia.org/wiki/Intelligent_agent
tags:
  - encyclopedia
status: complete
importance: medium
---

# Intelligent agent (Wikipedia)

**Source**: `sources/intelligent-agent-wikipedia.md` ([en.wikipedia.org/wiki/Intelligent_agent](https://en.wikipedia.org/wiki/Intelligent_agent), revision current on 2026-10-06)
**Type**: a secondary source standing in for Russell & Norvig's *Artificial Intelligence: A Modern Approach* (AIMA), which is not freely available. It cites AIMA's 2nd edition (2003, chapter 2) and 4th edition (2021).

## Summary

**The classical definition**, after Russell and Norvig: AI is "the study of agents that receive percepts from an environment and perform actions." An agent "perceives its environment through sensors and acts upon that environment through actuators." A **rational agent** chooses the action expected to maximize its **performance measure**, given its percept sequence and prior knowledge. Rational does not mean omniscient or always successful.

The article frames goals as an **objective function** (or utility function or reward function), with reinforcement learning as the case where a reward function is maximized.

## Russell & Norvig's five agent classes

The five classes, in increasing capability:
1. **Simple reflex agents**: condition-action rules ("if condition, then action") on the *current* percept only. They work only in fully observable environments and can loop forever in partially observable ones. Example: a thermostat.
2. **Model-based reflex agents**: keep internal state, a *model* of "how the world works", to handle partial observability.
3. **Goal-based agents**: add goal information and choose actions that reach desirable states. They need search and planning. The article lists "ChatGPT and the Roomba vacuum" as examples.
4. **Utility-based agents**: replace goal/not-goal with a **utility function** over states and maximize *expected* utility.
5. **Learning agents**:
   - a **learning element** improves a **performance element**;
   - a **critic** gives feedback on how well the agent is doing;
   - a **problem generator** suggests exploratory actions.
   - The article notes that the performance element "also called the '**actor**'" is adjusted using the critic's feedback. That is the same actor/critic vocabulary as RL's [[actor-critic]] methods.

## Other material in the article

- **Weiss's (2013) classification:** logic-based, reactive, belief–desire–intention (BDI), and layered architectures.
- **Hierarchies:** agents organized into "sub-agents" that form multi-agent systems.
- **Agentic AI:** in generative AI, "AI agents" (also called compound AI systems) operate autonomously over extended periods. The article notes "AI agents do not have a standard definition."

## Why it matters for the wiki

This is the classical frame the LLM agent literature inherits, usually without saying so:
- the [[react|ReAct]] loop is a percept → action cycle;
- tool calls are actuators, and tool results are percepts;
- the agent's [[world-model|world model]] maps to the model-based agent;
- [[verifier]]s and critics map to the learning agent's critic.

See [[intelligent-agent]] for the synthesis, and [[aima-ch2-intelligent-agents-litman-pitt]] for PEAS and environment types.

## Concepts & entities

[[intelligent-agent]] · [[actor-critic]] · [[reinforcement-learning]] · [[world-model]] · [[planning]] · [[multi-agent]] · [[llm-agent]] · [[artificial-intelligence]]
