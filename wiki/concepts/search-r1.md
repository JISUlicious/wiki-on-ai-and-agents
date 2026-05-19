---
title: Search-R1
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - search-r1-jin-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Search-R1

**Search-R1** is a reinforcement-learning recipe that trains an LLM to autonomously interleave reasoning with multi-turn calls to a real search engine, end-to-end, using only an outcome reward. It extends the [[deepseek-r1]]-Zero minimalism (no SFT warm-up, no process reward, no learned reward model) from pure parametric reasoning to *search-augmented* reasoning, and has become the seminal recipe cited as the foundation of [[agentic-rl]] on tool-call traces.

## Core mechanism

- **Retrieved-token loss masking** — the PPO/GRPO objective is computed only over LLM-generated tokens via an indicator `I(y_t)=1`; gradient is never flowed through retrieved passages. The mask applies to both the policy-gradient term *and* the KL term, and is the load-bearing trick that makes multi-turn RL stable.
- **Outcome-only reward (exact match)** — the rule-based reward is exact-match against the gold answer; no format reward, no process reward, no neural reward model. Format adherence emerges from the rollout structure itself.
- **Multi-turn interleaved rollout** — the model emits `<think>...</think>` blocks for reasoning and `<search>query</search>` blocks that trigger real retrieval; results are spliced back as `<information>...</information>` tokens, and a final `<answer>...</answer>` block terminates the trajectory.
- **PPO and GRPO both work** — PPO (actor-critic with value LLM and GAE) is the default in main results; GRPO (group-relative baseline, no critic) is evaluated as an alternative — the recipe is RL-algorithm-agnostic.
- **+41% on Qwen2.5-7B / +20% on Qwen2.5-3B** — averaged across seven QA benchmarks (NQ, TriviaQA, PopQA, HotpotQA, 2WikiMultiHopQA, Musique, Bamboogle) over [[retrieval-augmented-generation|RAG]] baselines under matched retriever, corpus, and training data. The model emergently learns to dynamically adjust retrieval frequency by problem complexity.

## What it advances

Search-R1 made [[agentic-rl]] practical by identifying the precise failure mode of naive multi-turn RL — gradient flow through retrieved context — and giving a one-line fix (mask retrieved tokens) that requires no architecture change. It directly seeded a wave of follow-up "X-R1" recipes (Tool-R1, Agent-R1, Graph-R1, DeepResearcher) that apply the same template to other tool ecosystems.

## Relation to other concepts

- Direct successor to [[deepseek-r1]]-Zero, extended from parametric reasoning to retrieval-augmented reasoning.
- Outperforms [[retrieval-augmented-generation]] baselines, IRCoT, Search-o1, and inference-only ReAct-style prompting.
- Uses [[ppo]] by default with [[grpo]] as an evaluated alternative.
- The seminal recipe for [[agentic-rl]] on tool-call traces; cited as ancestor by most 2025 RL-for-agents papers.
- Search is treated as the canonical [[tool-use]] action; the recipe is expected to generalize beyond retrieval.

## References

- [[search-r1-jin-2025]] — COLM 2025 paper; [arXiv:2503.09516](https://arxiv.org/abs/2503.09516)
