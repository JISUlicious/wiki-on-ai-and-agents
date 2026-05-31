---
title: Skill Acquisition
type: concept
created: 2026-05-29
updated: 2026-05-29
sources:
  - agent-skills-survey-xu-2026.md
status: draft
importance: medium
tags:
  - 2026
---

# Skill Acquisition

**Skill acquisition** is the question of *how* an agent comes to possess the [[agent-skills|skills]] it can later invoke. The acquisition taxonomy from the [[agent-skills-survey-xu-2026|Xu & Yan 2026 survey]] distinguishes several routes by which a skill enters an agent's repertoire, ranging from fully human-authored to fully autonomous. The choice of route shapes how reliable, composable, and verifiable the resulting skills are.

- **Reinforcement-learning-trained** — skills emerge as policies optimized against task reward (see [[reinforcement-learning]]).
- **Autonomous discovery** — the agent proposes and learns skills on its own during open-ended interaction (e.g. [[voyager]] in Minecraft).
- **Compositional synthesis** — new skills are assembled by composing or refactoring existing ones ([[library-learning]], [[tool-creation]]).
- **Human-authored** — skills are written and packaged by people (the `SKILL.md` style of [[agent-skills]]).

These routes are not mutually exclusive; a [[self-improving-agent]] may bootstrap from human-authored seeds and then discover and compose new skills autonomously.

## Related

- [[agent-skills]] — what is being acquired.
- [[voyager]] — canonical autonomous-discovery example.
- [[self-improving-agent]] — agents that acquire skills over their lifetime.

## References

- [[agent-skills-survey-xu-2026]]
