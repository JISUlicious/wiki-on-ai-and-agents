---
title: Memory-R1
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - memory-r1-yan-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Memory-R1

Memory-R1 (Yan et al., 2025) is a reinforcement-learning framework that turns the **memory lifecycle itself** — what to add, update, delete, or leave alone — into a *learned* policy rather than a prompted heuristic. It adopts the `{ADD, UPDATE, DELETE, NOOP}` operator set from [[mem0]] but fine-tunes a dedicated Memory Manager agent with [[ppo]] or [[grpo]] using only downstream QA exact-match as the reward signal. A second Answer Agent learns *memory distillation* (filtering ~60 retrieved candidates before reasoning). With only **152 training QA pairs**, Memory-R1-GRPO delivers +28% F1, +34% BLEU-1, +30% LLM-as-Judge over Mem0 on LoCoMo, generalizing zero-shot to MSC and LongMemEval across 3B–14B backbones.

## Core mechanism

- **Two-agent decomposition**: a Memory Manager (writes/edits the memory bank) is decoupled from an Answer Agent (reads/distills memories for QA); each is trained independently with its own RL loss.
- **Four discrete operations**: the Memory Manager outputs `(operation, content')` with `operation ∈ {ADD, UPDATE, DELETE, NOOP}` — the [[mem0]] action space, but now policy-controlled.
- **Outcome-driven reward**: a Memory Manager's operations are graded *only* by whether the frozen Answer Agent then answers the downstream question correctly (`R = EM(y_pred, y_gold)`); no human labels on individual edits.
- **PPO and GRPO variants**: both tested; GRPO consistently wins (no value-function critic, group-relative advantages over G sampled candidate operations).
- **Learned eviction**: DELETE is no longer a TTL or capacity rule — it is a policy action selected when removing an entry improves downstream answers. The motivating example: vanilla managers fragment "I adopted Buddy" + later "I adopted Scout" into DELETE+ADD; the RL-trained manager learns a single UPDATE consolidating both facts.

## What it advances

- [[memory-management]]: first framework to treat memory operation selection as an RL problem, not as in-context prompting.
- **Learned eviction policy**: DELETE becomes a graded action — forgetting is *learnable*, not engineered.
- **Memory distillation**: Answer Agent learns to filter retrieved noise before reasoning, addressing the "too many entries, model distracted" failure mode of vanilla [[retrieval-augmented-generation|RAG]].

## Relation to other systems

- **vs. [[mem0]] / [[a-mem]] / [[memgpt]]**: same or smaller operator set, but Memory-R1 supplies a real learning signal where prior systems rely on a vanilla LLM following in-context instructions. The *learned-vs-heuristic* axis is the core distinction.
- **vs. [[mem-alpha]]**: both use RL (GRPO) over memory operations; Memory-R1 trains an *editor* over a flat existing memory of facts (ADD/UPDATE/DELETE), while Mem-α trains a *constructor* that builds a multi-component (core/episodic/semantic) memory from scratch. The *editing-vs-construction* axis distinguishes them.
- **vs. retrieval-stage RL (e.g., Search-R1)**: Memory-R1 composes naturally with retrieval-stage RL — its Answer Agent already does retrieval-stage distillation; the open thread is end-to-end RL across both write and read policies.

## References

- Source: [[memory-r1-yan-2025]]
