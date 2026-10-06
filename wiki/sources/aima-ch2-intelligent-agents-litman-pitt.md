---
title: "Intelligent Agents: AIMA Chapter 2 (lecture handout, University of Pittsburgh)"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - aima-ch2-intelligent-agents-litman-pitt.md
authors:
  - Diane Litman
year: 2026
venue: "University of Pittsburgh CS 2710 lecture handout, after Russell & Norvig, AIMA 2nd ed."
url: https://people.cs.pitt.edu/~litman/courses/cs2710/lectures/ch02RN-handout.pdf
tags:
  - lecture-notes
status: complete
importance: medium
---

# Intelligent Agents: AIMA Chapter 2 (lecture handout)

**Source**: `sources/aima-ch2-intelligent-agents-litman-pitt.md` ([PDF](https://people.cs.pitt.edu/~litman/courses/cs2710/lectures/ch02RN-handout.pdf))
**Author**: course slides for Pitt CS 2710 (Diane Litman's course page), explicitly "after Russell and Norvig", AIMA 2nd ed. The handout itself is undated.
**Why this source**: it is a faithful secondary rendering of AIMA chapter 2. It covers **PEAS** and the **environment dimensions**, which the [[intelligent-agent-wikipedia|Wikipedia article]] omits.

## Key content

**Agents and environments.**
- An agent perceives through **sensors** and acts through **actuators**. For a softbot, these are keystrokes and displays.
- The **agent function** maps any *percept sequence* to an action. The **agent program** implements it.
- Example: the Reflex-Vacuum-Agent ("if dirty then suck, else move to the other square").

**Rationality.**
- A **performance measure** is "an objective, numerical value for any environment history." It is "generally better to measure what you want in the environment, rather than how you think the agent should behave."
- A rational agent selects the action expected to maximize the performance measure, given its percept sequence and built-in knowledge.
- Rational ≠ omniscient, clairvoyant or successful.
- Rationality implies information gathering (exploration), learning and **autonomy**: relying on percepts rather than on possibly wrong prior knowledge.

**PEAS: specifying the task environment.** "Task environments: 'problems' to which 'agents' are solutions."

| | Automated taxi driver (the AIMA example) |
|---|---|
| **P**erformance measure | correct destination, safe, fast, legal, comfortable, profitable |
| **E**nvironment | roads, traffic, pedestrians, customers |
| **A**ctuators | steering, accelerator, brake, horn |
| **S**ensors | camera, sonar, speedometer, GPS |

The handout adds a text-based conversational tutor: performance = test score; environment = students and the testing agency; actuators = displayed exercises, suggestions and corrections.

**Environment dimensions.**
- fully vs **partially observable**;
- deterministic vs **stochastic**;
- episodic vs **sequential**;
- static vs **dynamic** (and semidynamic);
- discrete vs **continuous**;
- single vs **multi-agent**, which may be cooperative or competitive.

**Agent types.** "Four basic types in order of increasing generality": simple reflex, model-based reflex with state, goal-based, and utility-based. "All these can be turned into **learning agents**." In a learning agent the earlier agent becomes the *performance element*, and a *problem generator* drives exploration.

## Mapping to LLM agents (wiki synthesis, not from the handout)

| AIMA | LLM agent |
|---|---|
| Performance measure | Task success metric or benchmark; reward |
| Environment | Repo, browser, OS, user, other agents |
| Actuators | Tool calls, code edits, messages |
| Sensors | Tool results, observations, retrieved documents |
| Agent function / program | Model plus harness |

**Environment type of a typical coding agent:** partially observable, stochastic, sequential, dynamic, discrete and often multi-agent. That is close to AIMA's hardest case.

## Concepts & entities

[[intelligent-agent]] · [[llm-agent]] · [[agent-computer-interface]] · [[multi-agent]] · [[reinforcement-learning]]
