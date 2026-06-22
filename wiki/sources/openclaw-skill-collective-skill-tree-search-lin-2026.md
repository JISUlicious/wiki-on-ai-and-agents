---
title: "OpenClaw-Skill: Collective Skill Tree Search for Agentic Large Language Models"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - openclaw-skill-collective-skill-tree-search-lin-2026.pdf
arxiv_id: "2606.16774"
authors:
  - Tianyi Lin
  - et al.
first_author: Tianyi Lin
year: 2026
tags: [2026]
status: complete
importance: medium
---

# OpenClaw-Skill: Collective Skill Tree Search for Agentic Large Language Models

*Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-14–06-21).*

## Summary

OpenClaw-Skill targets the problem of equipping LLM agents with reusable [[agent-skills]] for tool use, multi-step reasoning, and dynamic environment interaction in systems like [[openclaw]]. Rather than distilling one trajectory at a time, it proposes **Collective Skill Tree Search (CSTS)**, a tree-search-based [[skill-acquisition]] framework that constructs a structured, diverse, and generalizable *tree* of skills using collective intelligence from multiple models.

CSTS runs two iterative phases: Collective Skill Node Generation (CSN-Gen) explores diverse candidate skills per subtask by pooling knowledge across models, and Collective Skill Node Assessment (CSN-Assess) uses multiple models as judges with two scoring mechanisms — collective quality scoring (aggregated independent evaluations) and collective transferability scoring (verifies a skill generalizes across models). The output is a skill tree plus skill-augmented training data. A second stage, Collective Skill Reinforcement Learning (CSRL), trains agents to actively retrieve and apply multiple relevant skills from the hierarchy, broadening solution-space exploration and avoiding single-skill homogeneous solutions.

## Key points

- Builds a skill **tree** (not single-trajectory distillation) via two phases: CSN-Gen for diverse candidate generation, CSN-Assess for multi-judge selection.
- Two scoring mechanisms in CSN-Assess: collective **quality** scoring (robust effectiveness estimate) and collective **transferability** scoring (cross-model generalization check).
- CSRL selects multiple relevant skills from the tree at inference/training to avoid being trapped by a single suboptimal skill.
- On QwenClawBench, overall score gains of +5.8 (Qwen3-4B), +4.3 (Qwen3-8B), +9.7 (Qwen3.5-4B), and +10.4 (Qwen3.5-9B); e.g. OpenClaw-Skill 9B raises SVM 33.2→70.9 and CS 30.2→78.4.
- On PinchBench (23-task and 123-task settings), consistent gains, e.g. avg score 47.1→53.6 on the 123-task setting for the 9B model.
- Backbones: Qwen3-4B/8B and Qwen3.5-4B/9B; gains largest on long-horizon tool use and execution-feedback categories.

## Concepts & entities

- [[agent-skills]]
- [[skill-acquisition]]
- [[openclaw]]
- [[multi-agent]]
- [[reinforcement-learning]]

## References

- [arXiv:2606.16774](https://arxiv.org/abs/2606.16774)
