---
title: Agentic RL
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - deepseek-r1-2025.md
  - search-r1-jin-2025.md
status: complete
importance: high
tags:
  - 2025
---

**Agentic RL** is the umbrella term for training LLM-based agents end-to-end with reinforcement learning over **multi-step reasoning and tool-use trajectories**, optimizing for the final outcome of a long-horizon task rather than for single-turn preferences. Where classic [[rlhf]] elicits chat behaviors by fitting a reward model to human preference pairs over short completions, and where supervised tool-use fine-tuning imitates curated demonstrations, agentic RL lets the agent generate its own rollouts in an environment (a search engine, a shell, a browser, a code sandbox, a GUI), assigns reward only at the trace level (correctness, test-pass, task-success), and propagates that signal through the entire interleaved think-act-observe sequence. The 2025 inflection was [[deepseek-r1]] demonstrating that this works on pure parametric reasoning with rule-based outcome rewards; [[search-r1]] then established the canonical recipe for tool-use, and a wave of follow-ups extended it to coding, computer use, memory, and research agents.

## Core mechanism

- **Outcome-based reward**: terminal scalar from the environment (exact-match answer, unit-test pass, benchmark resolved-rate) replaces learned preference models; this is the load-bearing simplification inherited from [[deepseek-r1]]-Zero and is explicitly chosen to avoid reward hacking on long traces.
- **GRPO over PPO**: [[grpo]] (Group Relative Policy Optimization) drops the value network and computes advantages by normalizing rewards within a group of sampled rollouts, making long-rollout RL tractable at scale. Both [[ppo]] and GRPO remain in use; UI-TARS-2 stabilizes PPO with Decoupled-GAE for very long horizons.
- **Retrieved/observation token masking**: policy-gradient losses are computed only over LLM-generated tokens, never over tool outputs spliced back into the context — without this masking, gradient flow through retrieved or observed content destabilizes training ([[search-r1]] §3.1).
- **Environment-in-the-loop rollouts**: the rollout alternates `<think>` generation with action calls (`<search>`, click, edit, shell) until an answer block is emitted or a budget is exhausted; the environment is part of the MDP rather than a differentiable layer.
- **Emergent multi-step behaviors**: self-verification, backtracking, dynamic retrieval frequency, and growing CoT length appear without explicit process supervision — the central empirical claim of the agentic-RL paradigm.

## What it advances

Agentic RL is the post-training paradigm that turned LLMs from one-shot responders into trace-optimized agents. It made long-horizon tool-use, multi-hop QA, GUI control, and SWE-agent work measurable and improvable with a single scalar reward, decoupling agent capability from the quality of demonstration data. The open-weights release of DeepSeek-R1 in January 2025 made the recipe available to anyone, and the field rapidly produced Search-R1, Tool-R1, Agent-R1, Graph-R1, [[memory-r1]], [[mem-alpha]], DeepResearcher, and [[ui-tars-2]] — all variations on the same outcome-reward + masked-rollout template applied to a new action surface.

## Relation to other concepts

Sits adjacent to [[rlhf]] (preference-fitting on short chats) and [[constitutional-ai]] (rule-based preferences), but optimizes a different objective: long-trace task success. Subsumes the tool-use-RL recipe of [[search-r1]] and the reasoning-RL recipe of [[deepseek-r1]]. Compose with [[grpo]] and [[ppo]] as optimizers, with [[chain-of-thought]] as the surface form inside `<think>`, and with [[swe-gym]] / [[swe-bench-live]] as the training and evaluation surfaces for coding agents. Compare to [[self-improving-agent]] approaches like [[darwin-godel-machine]] which evolve scaffolds without weight updates.

## References

- [[deepseek-r1-2025]] — open-weights inflection point; R1-Zero pure-RL recipe.
- [[search-r1-jin-2025]] — canonical tool-use-RL formulation with retrieved-token masking.
- [[ui-tars-2-wang-2025]] — multi-turn agentic RL for computer-use at scale.
- [[memory-r1-yan-2025]], [[mem-alpha-wang-2025]] — agentic RL applied to memory management.
