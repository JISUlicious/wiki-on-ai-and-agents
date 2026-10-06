---
title: "Reinforcement Learning: An Introduction (2nd ed.), Chapter 13: Policy Gradient Methods"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - rl-an-introduction-sutton-barto-2020.md
authors:
  - Richard S. Sutton
  - Andrew G. Barto
introduces:
  - "[[actor-critic]]"
year: 2018
venue: "MIT Press, 2nd edition (2018); free PDF dated 2020"
url: http://incompleteideas.net/book/RLbook2020.pdf
tags:
  - book
status: complete
importance: high
---

# Reinforcement Learning: An Introduction, Chapter 13

**Source**: `sources/rl-an-introduction-sutton-barto-2020.md`, extracted from the [RLbook2020.pdf](http://incompleteideas.net/book/RLbook2020.pdf) (548 pages). Only **chapter 13** (Policy Gradient Methods, pp. 321–338), its *Bibliographical and Historical Remarks*, and the ANN history in §9.10 were read for this page.
**Authors**: Richard S. Sutton and Andrew G. Barto
**Date**: 2nd edition 2018 (MIT Press); this PDF is the 2020 version

## Chapter 13 in brief

The chapter covers methods that learn a **parameterized policy** π(a|s, θ) directly, rather than deriving actions from value estimates:
- **Advantages:** they can learn stochastic policies, handle continuous actions, and approach deterministic policies gradually.
- **Policy gradient theorem:** an exact expression for the gradient of performance with respect to θ that does not involve the derivative of the state distribution.
- **REINFORCE:** a Monte Carlo policy gradient. Adding a **baseline** (a learned state value) reduces variance without adding bias.
- **[[actor-critic|Actor–critic]] methods (§13.5).**
  - When the state-value function is also used to assess the action, through the one-step return R + γv̂(S′) − v̂(S), "it is called a **critic**, and the overall policy-gradient method is termed an **actor–critic method**."
  - The critic adds bias but greatly reduces variance. The book compares this with bootstrapping TD methods beating Monte Carlo methods.
  - Variants: one-step, eligibility traces, continuing problems, and continuous actions.
- **Summary:** "If the state-value function is also used to assess—or criticize—the policy's action selections, then the value function is called a critic and the policy is called an actor."

## Origin (Historical Remarks and §9.10)

- Methods "related to policy gradients were actually some of the earliest to be studied in reinforcement learning": Witten 1977; **Barto, Sutton & Anderson 1983**; Sutton 1984; Williams 1987 and 1992.
- **The first actor–critic:** "Barto, Sutton, and Anderson (1983) presented an actor–critic algorithm in the form of an ANN learning to balance a simulated pole." The paper is "Neuronlike elements that can solve difficult learning control problems" (IEEE Transactions on Systems, Man, and Cybernetics, 13(5), 1983), on the cart-pole task.
- These methods were "largely supplanted in the 1990s by action-value methods", then returned to favour.
- The term **"policy gradient methods"** was introduced by Sutton, McAllester, Singh & Mansour (2000).
- Later developments: natural gradients, deterministic policy gradients (Silver et al. 2014), off-policy variants and entropy regularization. Applications include AlphaGo.

## Lineage to LLMs (wiki synthesis)

- **[[ppo|PPO]]** (2017) is an actor–critic policy-gradient method. It is the optimizer of [[rlhf|RLHF]], in [[instructgpt-ouyang-2022|InstructGPT]]. In RLHF the "critic" is a value head, separate from the [[reward-modeling|reward model]].
- **GRPO** ([[deepseek-r1]], 2025) drops the learned critic and uses a group-relative baseline. In effect it returns from actor–critic to REINFORCE with a baseline.
- **The metaphor travels beyond RL:**
  - a learned evaluator criticizes a generator's outputs, in [[verifier]]s and [[llm-as-a-judge|LLM judges]];
  - the actor/critic vocabulary also appears in AIMA's learning agent (see [[intelligent-agent]]);
  - Anthropic's [[agentic-workflow-patterns|evaluator-optimizer]] workflow and [[reflexion|Reflexion]] use the same split.
  - These are analogies, not the algorithm: nothing is trained by policy gradient in a judge loop.

## Concepts & entities

[[actor-critic]] · [[reinforcement-learning]] · [[ppo]] · [[rlhf]] · [[reward-modeling]] · [[verifier]] · [[deepseek-r1]] · [[intelligent-agent]]
