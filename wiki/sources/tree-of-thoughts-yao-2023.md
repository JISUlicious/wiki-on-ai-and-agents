---
title: "Tree of Thoughts: Deliberate Problem Solving with Large Language Models"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - yao-tot-2023.md
arxiv_id: "2305.10601"
venue: NeurIPS 2023
authors:
  - "[[shunyu-yao]]"
  - Dian Yu
  - Jeffrey Zhao
  - Izhak Shafran
  - Thomas L. Griffiths
  - Yuan Cao
  - Karthik Narasimhan
first_author: "[[shunyu-yao]]"
introduces:
  - "[[tree-of-thoughts]]"
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# Tree of Thoughts: Deliberate Problem Solving with Large Language Models

**Source**: `sources/yao-tot-2023.md` (extracted from `yao-tot-2023.pdf`)
**arXiv**: [2305.10601](https://arxiv.org/abs/2305.10601)
**Venue**: NeurIPS 2023
**Authors**: [[shunyu-yao]] ([[princeton-university|Princeton]]; also first author of [[react-yao-2022|ReAct]]), Dian Yu, Jeffrey Zhao, Izhak Shafran (Google DeepMind), Thomas L. Griffiths (Princeton), Yuan Cao (Google DeepMind), Karthik Narasimhan (Princeton)

## Summary

Tree of Thoughts (ToT) generalizes [[chain-of-thought-prompting|chain-of-thought prompting]] from a **linear chain** of reasoning into a **search tree**. At each step, the LLM proposes multiple candidate "thoughts" (intermediate reasoning steps). The system then self-evaluates the candidates, expands promising branches, and **backtracks** when a branch reaches a dead end — implementing classical AI search (BFS, DFS, or value-guided search) over thought space, with the LLM playing the role of expansion and evaluation operator.

The framing draws on dual-process theory (Kahneman): standard left-to-right LLM decoding is "System 1" (fast, automatic); ToT adds "System 2" deliberation via explicit search. Three components:

1. **Thought decomposition**: define what counts as a "thought" for the task (e.g., one line of crossword fill, one arithmetic step, one paragraph of a creative writing plan).
2. **Thought generator**: the LLM proposes K candidate next thoughts given the current state.
3. **State evaluator**: the LLM scores intermediate states (e.g., "this partial solution is promising / likely a dead end") — used to prune the search.

Empirical demonstrations on three tasks requiring planning / search:

- **Game of 24** (arithmetic): GPT-4 + greedy CoT solves 4%; GPT-4 + ToT solves **74%**.
- **Creative Writing** (4-paragraph coherent text from given endings): ToT produces more coherent results per human eval.
- **Mini Crosswords**: ToT solves 60% of letters vs. 16% for greedy CoT.

ToT is the conceptual bridge from "linear CoT reasoning" to "agent-style planning" — it makes the implicit search inside a single forward pass explicit, controllable, and inspectable. The technique inspired LATS (LLM Agent Tree Search), Reasoning with Language Model is Planning with World Model, MCTS-flavored LLM planning, and the broader test-time-search literature.

## Key Points

- **Search over thought space**, not just over tokens. Each node is a coherent unit of text (a "thought"), not a single token.
- **Components**:
  - Thought decomposition (task-specific).
  - Thought generator: sample or prompt the LLM to propose K candidate thoughts.
  - State evaluator: prompt the LLM to score intermediate states (or use a value function).
- **Search strategy**: BFS, DFS, or beam search over the tree. Backtrack on dead ends.
- **Result on Game of 24**: 4% (CoT) → 74% (ToT). Most dramatic published demonstration of an inference-time technique's effect on a planning task.
- **Cost**: roughly K× × depth more inference than CoT. Trade-off: pay more compute for problems CoT can't solve.

## Entities Mentioned

- [[shunyu-yao]] — first author (also [[react-yao-2022|ReAct]])
- [[princeton-university]] — author affiliation
- [[deepmind]] (Google DeepMind) — co-authors
- Karthik Narasimhan — Yao's PhD advisor at Princeton (and a co-author on [[gpt-1-radford-2018|GPT-1]] earlier in his career)

## Concepts Discussed

- [[tree-of-thoughts]] — the technique
- [[chain-of-thought-prompting]] — what it generalizes
- [[self-reflection]] — the state-evaluator step is a form of self-evaluation
- [[llm-agent]] — ToT-style search inside an agent loop is the natural next step

## Notable Quotes

> "We introduce a new framework for language model inference, 'Tree of Thoughts' (ToT), which generalizes over the popular 'Chain of Thought' approach to prompting language models, and enables exploration over coherent units of text ('thoughts') that serve as intermediate steps toward problem solving. ToT allows LMs to perform deliberate decision making by considering multiple different reasoning paths and self-evaluating choices ... as well as looking ahead or backtracking when necessary to make global choices." — Abstract

> "In Game of 24, while GPT-4 with chain-of-thought prompting only solved 4% of tasks, our method achieved a success rate of 74%." — Abstract

## References

_Original source: `sources/yao-tot-2023.md`_
_Code: https://github.com/princeton-nlp/tree-of-thought-llm_
