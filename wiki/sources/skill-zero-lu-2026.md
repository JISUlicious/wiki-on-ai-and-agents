---
title: "SKILL0: In-Context Agentic Reinforcement Learning for Skill Internalization"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - skill-zero-lu-2026.md
arxiv_id: "2604.02268"
authors:
  - Zhengxi Lu et al.
first_author: Zhengxi Lu
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# SKILL0: In-Context Agentic Reinforcement Learning for Skill Internalization

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

SKILL0 (Zhejiang University, Meituan, Tsinghua) targets a limitation of inference-time **agent skills**: retrieval noise injects irrelevant guidance, loaded skill content costs many tokens, and the model never truly *acquires* the knowledge it only follows. The authors ask whether skills can instead be **internalized into model parameters**, enabling zero-shot autonomous behavior without any runtime skill retrieval — summarized by the motto "Skills at training, zero at inference."

The method is an in-context reinforcement learning framework with a training-time **curriculum** that begins with full skill context and progressively withdraws it. Skills are grouped offline by category and rendered, with interaction history, into a compact context that teaches tool invocation and multi-turn task completion. A **Dynamic Curriculum** scores each skill file's on-policy helpfulness and retains only those the current policy still benefits from, under a linearly decaying budget, until the agent operates fully zero-shot. Across agentic benchmarks it beats the standard RL baseline (+9.7% ALFWorld, +6.6% Search-QA, +10.1% WebShop) while keeping context under 0.5k tokens per step.

## Key points

- Reframes skills from inference-time augmentation (retrieve-and-inject) to training-time **internalization** into weights.
- Curriculum starts with full skill context and linearly decays the skill budget toward zero-shot operation.
- Dynamic Curriculum prunes skill files by measured **on-policy helpfulness**, keeping only those that still help the current policy.
- Skills grouped offline by category, rendered with interaction history into a compact (visual) context teaching multi-turn tool use.
- Gains over RL baseline: +9.7% ALFWorld, +6.6% Search-QA, +10.1% WebShop; per-step context kept under 0.5k tokens.
- Positioned against skill-augmentation methods (e.g. SkillRL) and the Claude Code / OpenClaw scaffold ecosystem.
- Code released at github.com/ZJU-REAL/SkillZero.

## Concepts & entities

[[skill-acquisition]] · [[agent-skills]] · [[agentic-rl]] · [[reinforcement-learning]] · [[in-context-learning]] · [[skill-library]] · [[llm-agent]] · [[tool-use]] · [[zhejiang-university]]

## References

- arXiv: [arXiv:2604.02268](https://arxiv.org/abs/2604.02268)
- Code: https://github.com/ZJU-REAL/SkillZero
