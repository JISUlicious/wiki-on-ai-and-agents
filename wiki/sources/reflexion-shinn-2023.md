---
title: "Reflexion: Language Agents with Verbal Reinforcement Learning"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - reflexion-shinn-2023.md
arxiv_id: "2303.11366"
venue: NeurIPS 2023
authors:
  - Noah Shinn
  - Federico Cassano
  - Edward Berman
  - Ashwin Gopinath
  - Karthik Narasimhan
  - "[[shunyu-yao]]"
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# Reflexion: Language Agents with Verbal Reinforcement Learning

**Source**: `sources/reflexion-shinn-2023.md` (extracted from `Reflexion Shinn 2023 2303.11366.pdf`)
**arXiv**: [2303.11366](https://arxiv.org/abs/2303.11366)
**Venue**: NeurIPS 2023
**Authors**: Noah Shinn, Federico Cassano, Edward Berman (Northeastern), Ashwin Gopinath (MIT), Karthik Narasimhan, [[shunyu-yao]] ([[princeton-university|Princeton]])

## Summary

Reflexion is a framework for improving [[llm-agent|LLM agents]] across episodes **without updating any model weights**. After each attempt at a task, the agent generates a **natural-language reflection** on what went wrong (or right). The reflection is stored in an **episodic memory buffer** and prepended to the prompt in subsequent attempts. The result is "verbal reinforcement learning": the policy is parameterized by `(LLM weights, memory)`, and the only thing that updates between trials is the memory.

This dodges the central problem of reinforcement-learning LLM agents: traditional RL (PPO, etc.) requires huge sample counts and expensive fine-tuning, which is infeasible for 100B+ models making API calls. Reflexion sidesteps weight updates entirely and instead exploits the model's **self-evaluation** capability to convert thin reward signals into rich textual feedback.

The headline result: **Reflexion achieves 91% pass@1 on HumanEval, surpassing [[gpt-4|GPT-4]]'s reported 80%** (single-shot, no reflection). It also outperforms strong baselines on ALFWorld decision-making (+22% absolute over [[react|ReAct]]) and HotpotQA reasoning (+20%).

Reflexion exemplifies a broader pattern that became central to the agent / reasoning-model era: **using LLMs to evaluate, critique, and revise their own outputs**, with all the optimization happening in token space rather than weight space.

## Key Points

- **Framework**: agent runs an episode → environment returns feedback (binary, scalar, or LLM-generated) → agent's "self-reflection" generates a natural-language critique → critique stored in memory → next episode prepends memory.
- **Three flavors of evaluation signal**:
  1. **External binary** (e.g., did the unit tests pass?).
  2. **Pre-defined heuristics** (rules for common failure modes in a given environment).
  3. **LLM self-evaluation** (the model judges its own trajectory, e.g., via self-generated unit tests for code).
- **No weight updates**: ideal for use with closed-weight frontier LLMs accessed via API.
- **Advantages over PPO-style RL** per the paper:
  1. No fine-tuning.
  2. Richer feedback than scalar reward.
  3. Interpretable episodic memory.
  4. More explicit action hints for future episodes.
- **Disadvantages**:
  1. Relies on LLM self-evaluation quality.
  2. No formal convergence guarantees.
- **Results**:
  - HumanEval: 91% pass@1 (vs. GPT-4 baseline 80%).
  - ALFWorld decision-making: +22% absolute over baseline in 12 trials.
  - HotpotQA: +20% over baseline.
- **Contribution beyond Reflexion itself**: LeetcodeHardGym — 40 Leetcode "hard" problems in 19 languages, intended as an RL-style env for code generation.

## Entities Mentioned

- [[shunyu-yao]] — co-author (also first author of [[react-yao-2022|ReAct]])
- [[princeton-university]]
- [[gpt-4]] — baseline that Reflexion exceeds on HumanEval

## Concepts Discussed

- [[reflexion]] — the framework
- [[llm-agent]] — the broader paradigm
- [[react]] — the immediate predecessor baseline
- [[self-reflection]] — the central mechanism
- [[chain-of-thought-prompting]] — adjacent prompting paradigm
- [[humaneval]] — the headline benchmark

## Notable Quotes

> "We propose Reflexion, a novel framework to reinforce language agents not by updating weights, but instead through linguistic feedback." — Abstract

> "Reflexion achieves a 91% pass@1 accuracy on the HumanEval coding benchmark, surpassing the previous state-of-the-art GPT-4 that achieves 80%." — Abstract

## References

_Original source: `sources/reflexion-shinn-2023.md`_
_Code: https://github.com/noahshinn024/reflexion_
