---
title: Reflexion
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - reflexion-shinn-2023.md
status: complete
importance: high
---

# Reflexion

Reflexion is a framework for [[llm-agent|LLM-agent]] improvement across episodes via **verbal self-reflection** rather than weight updates. Introduced by [[reflexion-shinn-2023|Shinn et al. (2023)]].

## Loop

```
while not solved and attempts < budget:
    trajectory = agent.run(task, memory)
    feedback   = evaluate(trajectory)        # binary, scalar, or LLM judgment
    reflection = LLM.reflect(trajectory, feedback)
    memory.append(reflection)
```

The agent itself is just an LLM (e.g., [[react|ReAct]]-style). What evolves is the **textual memory** prepended to subsequent attempts.

## Why "verbal RL"

In standard RL, gradients update model weights based on a reward signal. In Reflexion, a **language summary** of why the previous episode failed is appended to the prompt for the next episode. The model's weights never change; the policy changes because the input (prompt) changes. It's an exploitation of the model's existing in-context-learning and self-criticism abilities.

## Sources of evaluation signal

1. **External binary**: e.g., did the unit tests pass? Did the agent reach the goal state?
2. **Heuristic**: pre-defined rules (e.g., "if you tried to pick up an object you weren't holding, that's a hallucination").
3. **LLM self-evaluation**: the LLM grades its own trajectory (sometimes via self-generated unit tests).

## Trade-offs

**Pros:**
- No fine-tuning — works with any LLM, especially closed-weight ones.
- Rich, targeted feedback (natural language carries more bits than a scalar).
- Interpretable: you can read the memory and see why the agent is behaving as it is.
- Episode-level optimization in seconds, not days.

**Cons:**
- Depends on LLM self-evaluation quality — confabulated reflections can mislead future episodes.
- No convergence guarantee.
- Memory can grow unwieldy without summarization / pruning.

## Lineage

Reflexion is part of a broader pattern of **self-critique / self-revision** prompting:

- Self-Refine (Madaan et al. 2023) — single-task iterative refinement, no memory.
- Reflexion — episodic memory across trials.
- Self-consistency (Wang et al. 2022) — multiple samples, vote on the answer.
- Tree-of-Thoughts (Yao et al. 2023) — explicit search over reasoning paths.
- Constitutional AI (Anthropic 2022) — LLM critiques itself against principles during training.
- Reasoning models (OpenAI o1, Claude reasoning, 2024–) — internal CoT/reflection learned via RL.

## References

- [[reflexion-shinn-2023]]
