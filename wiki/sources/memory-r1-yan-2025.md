---
title: "Memory-R1: Enhancing LLM Agents to Manage and Utilize Memories via RL"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - memory-r1-yan-2025.md
arxiv_id: "2508.19828"
authors:
  - Sikuan Yan
  - Xiufeng Yang
  - Zuchao Huang
  - Ercong Nie
  - Zifeng Ding
  - Zonggen Li
  - Xiaowen Ma
  - Jinhe Bi
  - Kristian Kersting
  - Jeff Z. Pan
  - Hinrich Schuetze
  - Volker Tresp
  - Yunpu Ma
first_author: Sikuan Yan
year: 2025
introduces:
  - "[[memory-r1]]"
  - "[[rl-memory-policy]]"
tags:
  - 2025
status: complete
importance: high
---

# Memory-R1: Enhancing LLM Agents to Manage and Utilize Memories via RL

- **Source**: `sources/memory-r1-yan-2025.md`
- **arXiv**: [arXiv:2508.19828](https://arxiv.org/abs/2508.19828)
- **Authors**: Sikuan Yan, Xiufeng Yang, Zuchao Huang, Ercong Nie, Zifeng Ding, Zonggen Li, Xiaowen Ma, Jinhe Bi, Kristian Kersting, Jeff Z. Pan, Hinrich Schuetze, Volker Tresp, Yunpu Ma (LMU Munich; Munich Center for Machine Learning; TU Munich; Cambridge; HKU; TU Darmstadt; Edinburgh)
- **Year**: 2025 (v5: Jan 2026)

## Summary

Memory-R1 turns the **memory lifecycle itself** — what to add, update, delete, or leave alone — into a learned reinforcement-learning policy rather than a heuristic. The framework adopts the `{ADD, UPDATE, DELETE, NOOP}` operator set popularized by [[mem0]] but, instead of letting a vanilla LLM pick operations from in-context instructions, fine-tunes a dedicated **Memory Manager** agent with outcome-driven RL. A second **Answer Agent** is trained to perform *memory distillation*: of the ~60 entries returned by similarity-based [[retrieval-augmented-generation|RAG]], it selects the truly relevant ones before reasoning over them. Both agents are optimized with [[ppo]] or [[grpo]], using only the downstream QA exact-match score as the reward signal — no per-operation labels.

The motivating failure mode (Figure 1) is telling: when a user says "I adopted a dog named Buddy" and later "I adopted another dog named Scout", a vanilla manager treats the second utterance as a contradiction and issues DELETE+ADD, fragmenting memory. The RL-trained manager learns to issue a single UPDATE that consolidates both facts into "Andrew adopted two dogs, Buddy and Scout." This is the paper's core claim: *when to forget, merge, or leave alone* can be learned from end-task reward, not engineered as rules.

Empirically, with only **152 training QA pairs**, Memory-R1 outperforms strong heuristic baselines ([[mem0]], [[a-mem]], MemoryOS, LoCoMo-RAG) across three multi-session benchmarks ([[locomo]], MSC, LongMemEval) and across model scales from 3B to 14B (LLaMA-3.1-8B-Instruct, Qwen-2.5 Instruct family). On LoCoMo with LLaMA-3.1-8B, Memory-R1-GRPO delivers relative improvements of **+28% F1, +34% BLEU-1, +30% LLM-as-Judge** over Mem0. The authors position this as the first work to frame memory operation selection (and memory filtering) as an RL problem for LLM agents, contrasting with the static, prompt-engineered control flow of [[mem0]], [[a-mem]], and [[memgpt]].

## Key Points

- **Four discrete memory operations**: the Memory Manager outputs `(operation, content')` where operation ∈ `{ADD, UPDATE, DELETE, NOOP}`. The action space is intentionally minimal-but-expressive, inherited from [[mem0]].
- **Two-agent decomposition**: Memory Manager (writes/edits the memory bank) is decoupled from the Answer Agent (reads/distills memories for QA). Each is trained independently with its own RL loss.
- **Outcome-driven reward**: the Memory Manager's operations are graded *only* by whether the frozen Answer Agent then answers the downstream question correctly (`R = EM(y_pred, y_gold)`). No human labels on individual ADD/UPDATE/DELETE decisions.
- **PPO and GRPO variants**: both [[ppo]] and [[grpo]] are tested. GRPO consistently wins on LoCoMo (avoids the value-function critic, uses group-relative advantages).
- **Learned eviction policy**: DELETE is no longer a heuristic timeout or capacity rule — it's a policy action selected when removing an entry improves downstream answers.
- **Memory Distillation by the Answer Agent**: from 60 RAG-retrieved candidates, the agent learns to filter noise before reasoning, addressing the "too many entries, model distracted" failure mode of vanilla RAG.
- **Extreme data efficiency**: 152 training QA pairs are enough to beat heuristic baselines, with zero-shot generalization to MSC and LongMemEval.
- **Cross-scale gains**: improvements hold across 3B / 7B / 8B / 14B backbones, suggesting the bottleneck for prior memory systems was the heuristic control loop, not raw model capacity.

## Method Detail

**Memory Manager.** Modeled as policy π_θ(o, m' | x, M_old) where x is the newly extracted fact and M_old is the current memory bank. For PPO, the standard clipped surrogate is applied to (o, m') jointly. For GRPO, G candidate (operation, content) pairs are sampled per state; each yields a reward via the frozen Answer Agent, and the group-standardized score becomes the advantage. A KL term to a reference policy regularizes drift.

**Answer Agent.** Given question q and 60 retrieved memories M_ret, it generates answer y. Trained with the same PPO/GRPO machinery, EM(y_pred, y_gold) as the reward. The "memory distillation" behavior emerges from the reward — the agent learns to mentally filter the retrieved set before answering.

**Training data.** LoCoMo train split (152 QA pairs). Memory-SFT — a supervised behavior-cloning variant using GPT-5-generated trajectories on the same architecture — is included as an ablation to isolate the contribution of RL vs. supervised imitation.

## Entities Mentioned

- [[ludwig-maximilian-university-of-munich]]
- [[technical-university-of-munich]]
- [[university-of-cambridge]]
- [[university-of-edinburgh]]
- [[mem0]] — primary baseline; Memory-R1 inherits its operator set
- [[a-mem]] — heuristic agentic-memory baseline
- [[memgpt]] — earlier CRUD-style memory system
- [[locomo]] — primary benchmark
- [[llama-3]] (LLaMA-3.1-8B-Instruct backbone)
- [[qwen-2-5]] (3B / 7B / 14B backbones)

## Concepts Discussed

- [[memory-r1]] — the framework introduced here
- [[rl-memory-policy]] — the new paradigm of RL-trained memory operations
- [[memory-management]] — what to add/update/delete/keep
- [[memory-eviction]] — DELETE as a learned action rather than a heuristic
- [[memory-distillation]] — filtering retrieved memories before reasoning
- [[reinforcement-learning]]
- [[ppo]]
- [[grpo]]
- [[llm-agent]]
- [[retrieval-augmented-generation]]
- [[multi-session-dialogue]]
- [[outcome-based-reward]]

## Notable Quotes

> "We present Memory-R1, a reinforcement learning (RL) framework that equips LLMs with the ability to actively manage and utilize external memory through two specialized agents: a Memory Manager that learns structured operations, including ADD, UPDATE, DELETE, and NOOP; and an Answer Agent that preselects and reasons over relevant entries."

> "Existing approaches mainly rely on vanilla LLMs to choose operations from in-context instructions without any learning signal tied to correctness... RL is the missing ingredient for adaptive memory in LLM agents. By optimizing outcome-based rewards, models can learn when to add, update, delete, or retain information."

> "With only 152 training QA pairs, Memory-R1 outperforms strong baselines and generalizes across diverse question types, three benchmarks (LoCoMo, MSC, LongMemEval), and multiple model scales (3B–14B)."

## Significance

Memory-R1 marks a paradigm shift in LLM-agent memory research: it is the **first system to treat the memory lifecycle as an RL policy** rather than as a prompted heuristic. Where [[mem0]] and [[a-mem]] still depend on a vanilla LLM following in-context instructions to choose memory operations, Memory-R1 supplies a real learning signal — and shows that even a tiny dataset (152 pairs) suffices to outperform the entire heuristic family. The most important conceptual contribution is the demonstration that **forgetting is learnable**: DELETE becomes a policy action graded by its effect on downstream answers, not a TTL or capacity rule. This opens a clear research thread on RL-trained memory controllers, and naturally composes with retrieval-stage RL (Search-R1) and tool-use RL (Toolformer, ReAct-style agents).

## References

_Original source: `sources/memory-r1-yan-2025.md`_
