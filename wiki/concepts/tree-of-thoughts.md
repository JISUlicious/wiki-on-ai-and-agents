---
title: Tree of Thoughts (ToT)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - yao-tot-2023.md
builds-on: "[[chain-of-thought-prompting]]"
status: complete
importance: high
---

# Tree of Thoughts (ToT)

Tree of Thoughts generalizes [[chain-of-thought-prompting|chain-of-thought prompting]] from a linear reasoning chain into a **search tree**, with the LLM playing the role of both the move generator and the position evaluator. Introduced by [[tree-of-thoughts-yao-2023|Yao et al. (2023)]] at Princeton + Google DeepMind. The framing borrows from dual-process theory: standard left-to-right LLM decoding is "System 1"; ToT adds explicit "System 2" deliberation.

## Components

1. **Thought decomposition** — define what counts as a "thought" for the task. Examples:
   - Game of 24: each thought = an arithmetic step that combines two numbers.
   - Crosswords: each thought = a candidate word for a clue.
   - Creative writing: each thought = a paragraph-level plan or draft.
2. **Thought generator** — prompt the LLM to produce K candidate next thoughts given the current state. K=3–5 typically.
3. **State evaluator** — prompt the LLM (or use a value function) to score each candidate state: "sure / maybe / impossible" or a numeric score. Used to prune unpromising branches.
4. **Search strategy** — BFS, DFS, or beam search over the tree of thoughts. Backtrack from dead ends; expand promising branches deeper.

## Empirical highlights

- **Game of 24** (4 numbers, +-×÷ to make 24): GPT-4 + greedy CoT solves **4%**; GPT-4 + ToT solves **74%**.
- **Mini Crosswords** (5×5): 60% letters correct with ToT vs. 16% with greedy CoT.
- **Creative Writing**: more coherent than greedy CoT per human evaluation.

## Why it matters

ToT makes the implicit "search" inside an LLM's forward pass **explicit, controllable, and inspectable**. It's the conceptual bridge from linear reasoning to agent-style planning:

- **LATS** (LLM Agent Tree Search, Zhou et al. 2023) combines ToT with [[react|ReAct]] for tool-using agents.
- **Reasoning with Language Model is Planning with World Model** (Hao et al. 2023): related; LM-as-world-model + MCTS.
- **Reasoning models** (OpenAI o1, Sep 2024; Anthropic reasoning Claude; DeepSeek R1, 2025): internalize tree-of-thoughts-style deliberation via RL training on long-form reasoning traces. The user sees a single answer; the model has done extensive internal search.

## Cost

ToT trades **K^depth × evaluator-calls** more inference for the gains. Expensive but bounded; the practical sweet spot is K=3–5, depth=3–5, with aggressive evaluator-driven pruning.

## References

- [[tree-of-thoughts-yao-2023]]
