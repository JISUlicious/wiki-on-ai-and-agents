---
title: "Memento-Skills: Let Agents Design Agents"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - memento-skills-let-agents-design-agents-memento-team-2026.md
arxiv_id: "2603.18743"
authors:
  - Memento-Team
first_author: Memento-Team
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Memento-Skills: Let Agents Design Agents

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

Memento-Skills is a generalist, continually-learnable LLM agent system that acts as an *agent-designing agent*: it autonomously constructs, adapts, and improves task-specific agents through experience, rather than relying on human-designed agents. It is built on a memory-based reinforcement learning framework with stateful prompts, where reusable skills — stored as structured markdown files — serve as persistent, evolving memory that encodes both behavior and context, letting the agent carry knowledge forward across interactions.

Starting from simple elementary skills (web search, terminal operations), the agent improves via a **Read–Write Reflective Learning** mechanism (from Memento 2): in the read phase a behavior-trainable skill router selects the most relevant skill given the current stateful prompt; in the write phase the agent updates and expands its skill library from new experience. This closed loop enables continual learning **without updating LLM parameters** — all adaptation is realized through the evolution of externalized skills and prompts. On the GAIA (General AI Assistants) benchmark and Humanity's Last Exam, the system shows sustained gains of 26.2% and 116.2% relative improvement in overall accuracy.

## Key points

- Frames the agent as an **agent designer**: designs agents end-to-end for new tasks instead of using hand-built agents.
- Skills are **structured markdown files** acting as persistent, evolving memory encoding behavior + context.
- **Read–Write Reflective Learning**: read = behavior-trainable skill router selects a skill; write = agent updates/expands the skill library from experience.
- Continual learning happens **without LLM weight updates** — adaptation lives entirely in externalized skills/prompts (memory-based RL).
- Bootstraps from elementary skills (web search, terminal) and iteratively refines them.
- Results: +26.2% relative on GAIA and +116.2% relative on Humanity's Last Exam; code released.

## Concepts & entities

[[skill-acquisition]] · [[self-improving-agent]] · [[agent-skills]] · [[llm-agent]] · [[multi-agent]] · [[memory-systems]] · [[reinforcement-learning]] · [[gaia-benchmark]] · [[humanitys-last-exam]] · [[code-as-harness]]

## References

- arXiv: [arXiv:2603.18743](https://arxiv.org/abs/2603.18743)
- Code: https://github.com/Memento-Teams/Memento-Skills
- Local source: `sources/memento-skills-let-agents-design-agents-memento-team-2026.md`
