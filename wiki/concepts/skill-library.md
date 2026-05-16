---
title: Skill Library
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - voyager-wang-2023.md
status: draft
importance: medium
tags:
  - 2023
---

# Skill Library

A **skill library** is a growing collection of executable code skills that an agent has learned and can retrieve by natural-language description. Introduced by [[voyager]] ([[voyager-wang-2023|Wang et al. 2023]]) for open-ended Minecraft play, each skill is a self-contained code function indexed by the embedding of its NL description. When the agent encounters a new task it retrieves the top-k most relevant skills and writes new code that *composes* them — by never overwriting prior skills, the system avoids the catastrophic forgetting that plagues weight-update approaches.

The skill library is the [[procedural-memory|procedural-memory]] ancestor of the modern `SKILL.md` packaging convention: a discoverable, descriptor-keyed bundle of executable behavior that an LLM can load on demand.

## Where it appears

- [[voyager]] — Minecraft lifelong-learning agent; canonical implementation.
- [[procedural-memory]] — the cognitive category the skill library realizes.
- [[mirix]] — procedural-memory role generalizes the skill-library pattern across domains.

## References

- [[voyager-wang-2023]]
