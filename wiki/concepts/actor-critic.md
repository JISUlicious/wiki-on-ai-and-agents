---
title: Actor–Critic
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - rl-an-introduction-sutton-barto-2020.md
  - intelligent-agent-wikipedia.md
status: complete
importance: high
---

# Actor–Critic

A [[reinforcement-learning]] method with two learned parts:
- the **actor**, a parameterized policy that chooses actions;
- the **critic**, a learned value function that judges those actions.

The critic's estimate of the one-step return, R + γv̂(S′) − v̂(S), replaces the full Monte Carlo return in the policy-gradient update. This adds some bias but cuts variance sharply ([[rl-an-introduction-sutton-barto-2020|Sutton & Barto]], §13.5).

## Origin

- **1983:** Barto, Sutton & Anderson, "Neuronlike elements that can solve difficult learning control problems" (IEEE Trans. SMC). An actor–critic implemented as a neural network learns to balance a simulated pole (cart-pole). Sutton & Barto give this as the first actor–critic algorithm.
- Related early policy-gradient work: Witten 1977; Sutton 1984; Williams 1987 and 1992 (REINFORCE).
- **1990s:** largely supplanted by action-value methods such as Q-learning.
- **2000:** Sutton, McAllester, Singh & Mansour introduce the term "policy gradient methods" and the policy gradient theorem. Marbach & Tsitsiklis obtained it independently.
- **2010s revival:** natural gradients, deterministic policy gradients (2014), entropy regularization. AlphaGo is among the applications.

## Into LLMs

| Step | Relation to actor–critic |
|---|---|
| [[ppo\|PPO]] (2017) | An actor–critic policy-gradient algorithm |
| [[rlhf\|RLHF]] / [[instructgpt-ouyang-2022\|InstructGPT]] (2022) | PPO with the LLM as actor and a **value head as critic**. The *reward* comes from a separate [[reward-modeling\|reward model]]. |
| GRPO / [[deepseek-r1\|DeepSeek-R1]] (2025) | **Removes the learned critic** and uses a group-relative baseline: closer to REINFORCE with a baseline. It is cheaper, and avoids training a value model for long reasoning. |

## The metaphor beyond RL

The actor/critic division of labour recurs throughout this wiki. Only the RL cases above are actor–critic in the technical sense; the rest are analogies.
- **Classical AI:** Russell & Norvig's learning agent has a *performance element*, which Wikipedia also calls the "actor", improved using feedback from a *critic* ([[intelligent-agent]]).
- **LLM systems:**
  - generator plus [[verifier]], from [[training-verifiers-math-word-problems-cobbe-2021|Cobbe 2021]] onward;
  - [[llm-as-a-judge|LLM judges]];
  - Anthropic's evaluator-optimizer workflow ([[agentic-workflow-patterns]]);
  - [[reflexion|Reflexion]]'s self-critique.

## Related

[[reinforcement-learning]] · [[ppo]] · [[rlhf]] · [[reward-modeling]] · [[verifier]] · [[intelligent-agent]] · [[deepseek-r1]]

## References

- [[rl-an-introduction-sutton-barto-2020]]
- [[intelligent-agent-wikipedia]]
