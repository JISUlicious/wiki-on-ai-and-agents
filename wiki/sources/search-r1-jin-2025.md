---
title: "Search-R1: Training LLMs to Reason and Leverage Search Engines with RL"
type: source
created: 2026-05-17
updated: 2026-05-17
sources:
  - search-r1-jin-2025.md
arxiv_id: "2503.09516"
venue: COLM 2025
authors:
  - Bowen Jin
  - Hansi Zeng
  - Zhenrui Yue
  - Jinsung Yoon
  - Sercan Ö. Arık
  - Dong Wang
  - Hamed Zamani
  - Jiawei Han
first_author: Bowen Jin
year: 2025
introduces:
  - "[[search-r1]]"
  - "[[agentic-rl]]"
tags:
  - 2025
  - paper
status: complete
importance: high
---

# Search-R1: Training LLMs to Reason and Leverage Search Engines with Reinforcement Learning

[arXiv:2503.09516](https://arxiv.org/abs/2503.09516) · COLM 2025 · Bowen Jin, Hansi Zeng, Zhenrui Yue, Jinsung Yoon, Sercan Ö. Arık, Dong Wang, Hamed Zamani, Jiawei Han ([[uiuc]], [[umass]], [[google]] Cloud AI Research)

## Summary

Search-R1 extends the [[deepseek-r1]]-Zero recipe of outcome-reward [[reinforcement-learning]] from pure parametric reasoning to **search-augmented reasoning**. The model is trained to autonomously interleave `<think>...</think>` reasoning with `<search>query</search>` calls to a real retrieval engine; retrieved passages are returned inside `<information>...</information>` tokens and the final answer is emitted between `<answer>...</answer>` tags. Training is done end-to-end with [[ppo]] or GRPO over the rollout, but two ingredients make it stable: (1) **retrieved-token loss masking** — the policy-gradient objective is computed only over LLM-generated tokens, never over the retrieved context that lands in the rollout; and (2) a **rule-based outcome reward** (exact-match against the gold answer) with no format reward and no learned reward model.

On seven QA benchmarks (NQ, TriviaQA, PopQA, HotpotQA, 2WikiMultiHopQA, Musique, Bamboogle), Search-R1 yields average relative improvements of **+41% on Qwen2.5-7B and +20% on Qwen2.5-3B** over [[retrieval-augmented-generation|RAG]] baselines under matched retriever, corpus, and training data. (The arXiv-v5 abstract states +24% for the 7B figure; the contributions section and headline framing in the paper use +41% — this summary preserves both numbers.) Gains hold for both base and instruction-tuned models, and larger models benefit more from learning to search. The paper has become the reference recipe for **agentic RL on tool-call traces**, directly seeding follow-ups like Tool-R1, Agent-R1, Graph-R1, and DeepResearcher.

## Key Points

- **Retrieved-token masking** — token-level PPO/GRPO losses are masked over retrieved spans via an indicator `I(y_t)=1` only when `y_t` is an LLM-generated token; without this, gradient flow through retrieved content destabilizes training. The mask is applied to both the policy-gradient term and the KL term.
- **Multi-turn interleaved rollout** — the rollout alternates generation and retrieval until either an `<answer>` block is emitted or a maximum action budget `B` is exhausted. Search calls are triggered by the model emitting `</search>`; the harness parses the query, hits the retriever, and splices the results back into the context.
- **Outcome-only reward** — exact-match against the gold answer; no process rewards, no format rewards, no neural reward model. The paper explicitly frames this as inheriting DeepSeek-R1-Zero's minimalism.
- **RL algorithm-agnostic** — works with both PPO (actor-critic with a value LLM and GAE) and GRPO (group-relative baseline, no critic). PPO is the default in main results; Section 5.1 compares the two.
- **Search engine as part of the environment** — the policy is formalized as `π_θ(· | x; R) = π_θ(· | x) ⊗ R`, embedding retrieval into the MDP rather than treating it as a differentiable layer.
- **Datasets** — trained on the merged train sets of NQ + HotpotQA; evaluated on 7 QA datasets covering general QA and multi-hop QA; in- vs. out-of-domain results both improve.
- **Retriever** — E5 over the 2018 Wikipedia dump (Karpukhin et al. 2020 corpus), top-3 passages per call, held constant across all baselines.
- **Models** — Qwen2.5-3B and Qwen2.5-7B, in both Base and Instruct variants. Base models fail to follow inference-only prompts (IRCoT, RAG) but RL-tune successfully under Search-R1.
- **Baselines beaten** — direct inference, [[chain-of-thought]], RAG, IRCoT, Search-o1, SFT, rejection sampling with search, and a no-retrieval RL baseline (R1-style) trained on the same data.
- **Behavioral emergence** — the model learns to issue multiple search queries within one trajectory and dynamically adjusts retrieval frequency by problem complexity, without any process-level supervision.

## Entities

- [[uiuc]] — Bowen Jin, Zhenrui Yue, Dong Wang, Jiawei Han (Department of Computer Science)
- [[umass]] — Hansi Zeng, Hamed Zamani (Center for Intelligent Information Retrieval)
- [[google]] — Jinsung Yoon, Sercan Ö. Arık (Google Cloud AI Research; see also [[google-research]])

## Concepts

- [[search-r1]] — the method this paper introduces
- [[agentic-rl]] — Search-R1 is widely cited as the seminal recipe for this paradigm
- [[reinforcement-learning]] — outcome-reward RL on multi-turn tool-call rollouts
- [[ppo]] — default optimizer; retrieved-token masking applied to the clipped surrogate
- [[grpo]] — group-relative variant evaluated as an alternative
- [[retrieval-augmented-generation]] — the baseline family Search-R1 outperforms
- [[tool-use]] — search is treated as the canonical tool; the recipe generalizes
- [[chain-of-thought]] — `<think>` segments instantiate CoT inside the search loop
- [[deepseek-r1]] — direct ancestor; Search-R1 is positioned as DeepSeek-R1-Zero extended to retrieval
- [[exact-match]] — reward signal

## Quotes

> "Search-R1 optimizes LLM reasoning trajectories with multi-turn search interactions, leveraging retrieved token masking for stable RL training and a simple outcome-based reward function." — Abstract

> "We model the search engine as part of the environment, enabling sampled trajectory sequences that interleave LLM token generation with search engine retrievals." — §1, Contributions

> "Search-R1 can be viewed as an extension of DeepSeek-R1 Zero, which primarily focuses on parametric reasoning, by introducing search-augmented RL training for enhanced retrieval-driven decision-making." — §1

> "Applying the same optimization to retrieved tokens can lead to unintended learning dynamics. To address this, we introduce loss masking for retrieved tokens, ensuring the policy gradient objective is computed only over LLM-generated tokens." — §3.1

> "We do not incorporate format rewards, as our learned model already demonstrates strong structural adherence." — §3.4

## References (selected)

- Guo et al. 2025 — [[deepseek-r1]] (ancestor recipe)
- Schulman et al. 2017 — PPO
- Shao et al. 2024 — GRPO (DeepSeekMath)
- Lewis et al. 2020 — original RAG
- Yao et al. 2023 — ReAct (search-as-tool prompting baseline)
- Schick et al. 2023 — Toolformer (SFT baseline for tool use)
- Trivedi et al. 2022 — IRCoT
- Li et al. 2025 — Search-o1
- Karpukhin et al. 2020 — DPR / Wikipedia 2018 corpus
- Wang et al. 2022 — E5 retriever
- Yang et al. 2024 — Qwen2.5
