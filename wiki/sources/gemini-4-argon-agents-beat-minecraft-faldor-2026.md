---
title: "A Team of Gemini 4 Argon Agents Beats Minecraft"
type: source
created: 2026-10-29
updated: 2026-10-30
sources:
  - gemini-4-argon-agents-beat-minecraft-faldor-2026.md
arxiv_id: "2610.22319"
authors:
  - Maxence Faldor
  - Satinder Singh
  - Roberta Raileanu
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# A Team of Gemini 4 Argon Agents Beats Minecraft

**Maxence Faldor, Satinder Singh, Roberta Raileanu** ([[deepmind|Google DeepMind]]). [arXiv:2610.22319](https://arxiv.org/abs/2610.22319), v1, cs.AI, 15 Oct 2026. A technical-report-style run description.

## Summary

Four instances of a **general-purpose Gemini 4 Argon coding agent**, not built for Minecraft, each control one player through **Mineflayer** (an open-source JavaScript bot library) and its pathfinding plugin. The world is a vanilla 1.20.4 survival server on **Easy**, from a random seed. The agents act only by writing and running JavaScript programs and reading the text output; they never see pixels. They coordinate through in-game chat and a memory with private and team-shared parts. Their only game instruction is a one-paragraph objective: defeat the Ender Dragon. There is no curriculum, skill library, human-written tool or route. A fifth agent, the **Observer**, watches for specification gaming and never plays.

## Key points

- **Dragon defeated 27 h 14 min after the first program**, with **2,783 programs**, 63,313 distinct lines of JS, 229 chat messages and **605 deaths**. The 605 deaths include 69 to the dragon, 67 drowning, 57 falls or void, and 47 lava. The run spans 6 server restarts.
- **The End took nearly 14 hours.** The breakthrough, about 21 h in, was one agent noticing that the ten crystals heal the dragon and that **dirt pillars** reach them without a pickaxe.
- **Emergent team behaviors.** Roles were proposed in chat within 10 minutes. One Mineflayer crafting-quirk helper appears in 655 programs. The agents kept 146 pieces of persistent state, including 42 helpers, one of which (a timeout guard) is referenced in 1,519 programs. Agents corrected false success claims within minutes.
- **Cost:** about 1.35B input tokens (91% cache hits) and 16M output tokens over about 24k calls, roughly **$520** at introductory list price ($1,040 standard) up to the win.
- The Observer and a manual log review found **no specification gaming and no server commands**.

The newsletter blurb (DeepMind, Mineflayer, 27h14m, 2,783 programs, 605 deaths, single run, Easy) matches the paper.

## What it does not show

- **A single run**, the first attempt. The authors say it cannot show how reliable the setup is.
- **Mineflayer is a strong oracle.** It reports blocks behind rock and the dragon's internal phase state, and it pathfinds. This is much easier than the pixel-and-keyboard setting of SIMA 2. The game is on Easy difficulty, where hunger cannot kill.
- **Prior knowledge.** Gemini knows Minecraft and Mineflayer from pretraining, so the run tests turning knowledge into code and coordination, not learning an unseen game.
- **No ablations.** There is no single-agent arm and no run without shared memory, so the paper cannot say what the team structure contributed.
- Two community dragon kills came earlier: a single Claude Opus 5 agent with 37 high-level tools on Normal, and a planner on a fixed seed on Peaceful.

## Concepts & entities

- [[voyager]] · [[voyager-wang-2023]]: the Mineflayer-programs lineage. Voyager needed a curriculum and a [[skill-library]]; here the skill library is emergent (shared helpers in memory).
- [[code-as-policies]] · [[codeact]]: acting by writing programs.
- [[multi-agent]] · [[multi-agent-memory]] · [[embodied-agent]]
- [[reward-hacking]]: the Observer agent as a specification-gaming monitor.
- [[deepmind]]

## Related

- [[genie-4-new-generation-of-world-simulation-deepmind-2026]]: Gemini 4 Argon also serves as the task planner that feeds step captions to [[genie-4|Genie 4]] for robot simulations (Apptronik Apollo playing checkers).

## References

- [arXiv:2610.22319](https://arxiv.org/abs/2610.22319)
