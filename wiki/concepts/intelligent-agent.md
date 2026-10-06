---
title: Intelligent Agent (classical AI)
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - intelligent-agent-wikipedia.md
  - aima-ch2-intelligent-agents-litman-pitt.md
status: complete
importance: high
---

# Intelligent Agent (classical AI)

The textbook definition, after Russell & Norvig's *Artificial Intelligence: A Modern Approach* (AIMA). An **agent** perceives its environment through **sensors** and acts on it through **actuators**. A **rational agent** chooses the action expected to maximize its **performance measure**, given its percept sequence and built-in knowledge.

AIMA frames all of AI as the study of such agents. Its first edition was 1995 (general knowledge); the sources here cite the 2nd (2003) and 4th (2021) editions.

## PEAS: specifying the task

**P**erformance measure, **E**nvironment, **A**ctuators, **S**ensors. "Task environments [are] 'problems' to which 'agents' are solutions." The automated-taxi example:

| | Taxi |
|---|---|
| P | safe, fast, legal, comfortable, profitable, correct destination |
| E | roads, traffic, pedestrians, customers |
| A | steering, accelerator, brake, horn |
| S | camera, sonar, speedometer, GPS |

## Environment dimensions

- fully vs partially observable;
- deterministic vs stochastic;
- episodic vs sequential;
- static vs dynamic;
- discrete vs continuous;
- single vs multi-agent.

## Agent types (increasing generality)

1. **Simple reflex**: if-condition-then-action on the current percept only (a thermostat). Fails under partial observability.
2. **Model-based reflex**: keeps an internal model of the unseen world state.
3. **Goal-based**: chooses actions that reach desirable states, which requires search and [[planning]].
4. **Utility-based**: maximizes the *expected utility* of outcomes.
5. **Learning agent**: any of the above plus a *learning element*. A *critic* scores behaviour, and a *problem generator* proposes exploration. This is the same actor/critic vocabulary as [[actor-critic]].

## Mapping to LLM agents (wiki synthesis)

| Classical | LLM agent |
|---|---|
| Percepts / sensors | Tool results, observations, retrieved documents, user messages |
| Actuators | Tool calls, code edits, messages ([[agent-computer-interface]]) |
| Agent program | Model + harness ([[agent-three-layer-model]]) |
| Internal model | Context, memory and [[world-model]] |
| Goal / utility | Task instruction, `/goal` completion condition, reward |
| Critic | [[verifier]], [[llm-as-a-judge]], tests |
| Performance measure | Benchmark score; task success |

A [[react|ReAct]] agent is the classical percept → action loop with language as the internal state. A coding agent's environment is partially observable, stochastic, sequential, dynamic and often multi-agent, which is close to AIMA's hardest case.

Anthropic's split between *workflows* and *agents* ([[agentic-workflow-patterns]]) is the old distinction between control fixed by the designer and actions chosen from percepts and goals.

The [[intelligent-agent-wikipedia|Wikipedia article]] lists "ChatGPT and the Roomba vacuum" as goal-based agents. Many LLM agents arguably fit best as *model-based, goal-based learning agents*, with in-context rather than parametric learning.

## Other typologies

- Weiss (2013): logic-based, reactive, belief–desire–intention, layered.
- Hierarchies of sub-agents are multi-agent systems ([[multi-agent]], [[subagents]]).

## Related

[[llm-agent]] · [[artificial-intelligence]] · [[reinforcement-learning]] · [[actor-critic]] · [[planning]] · [[world-model]] · [[multi-agent]]

## References

- [[intelligent-agent-wikipedia]]
- [[aima-ch2-intelligent-agents-litman-pitt]]
