---
title: Voyager
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - voyager-wang-2023.md
status: complete
importance: high
tags:
  - 2023
---

# Voyager

Voyager is an open-ended embodied [[llm-agent]] for Minecraft that learns by accreting executable code into a persistent [[skill-library]]. It pairs a [[gpt-4]] backbone with three loops — automatic curriculum, skill retrieval, and iterative self-verifying code generation — to scale capability monotonically without parameter updates. Voyager is the canonical instantiation of *skill library = procedural memory* in the LLM-agent literature, and the load-bearing ingredient is the library itself: handing it to a different agent ([[autogpt]]) confers most of Voyager's transfer advantage.

## Core mechanism

- **Skill library as vector-indexed code store.** Each entry is `(NL description, embedding(description), executable JS program)`; new skills are appended, never overwritten, sidestepping catastrophic forgetting structurally.
- **Code as action space.** Skills `import` previously committed skills, so the library is compositional — `craftIronPickaxe` is synthesized from `craftStick`, `smeltIronIngot`, `makeFurnace`.
- **Automatic curriculum (in-context novelty search).** GPT-4 proposes the next task conditioned on inventory, biome, equipment, and task history, with the standing prompt "discover as many diverse things as possible."
- **Iterative prompting with three feedback channels.** Environment messages, JS interpreter errors, and a GPT-4 self-verification critic loop until the task is verified or the round cap (4) is hit.
- **Retrieval-then-generate.** When a new task is set, top-k skills retrieved by embedding similarity are spliced into the code-generation prompt as exemplars.

## What it advances

Voyager contributes to [[memory-management]] in the **write** and **curate** operations over [[procedural-memory]]:

- **Write**: a verified skill is committed to the library only after the self-verification critic accepts it — a quality gate on the write path.
- **Curate**: skills are content-addressed by description embedding; the library grows monotonically with no eviction, so curation is purely additive.
- **Retrieve**: embedding-similarity lookup over skill descriptions, returning compositional building blocks for the next code-generation pass.

Voyager has no consolidate or compact operation — the library is append-only by design, trading storage for forgetting resistance.

## Relation to other systems

- **vs. [[reflexion]]** — Reflexion stores natural-language self-critiques in episodic memory and replays them as context; Voyager stores *executable artifacts* in procedural memory and re-imports them. Reflexion improves the next attempt at the same task; Voyager grows a permanent capability.
- **vs. [[generative-agents]]** — Generative agents consolidate observations into abstract reflections (declarative memory); Voyager consolidates trial-and-error into reusable subroutines (procedural memory). Different memory subsystems, different consolidation primitives.
- **vs. [[react]] / [[autogpt]]** — These pick from a fixed tool set per episode and discard the trajectory. Voyager extends the tool set itself across episodes, which is why giving AutoGPT access to Voyager's library closes much of the gap.

## References

- Source: [[voyager-wang-2023]]
- arXiv: <https://arxiv.org/abs/2305.16291>
- Project: <https://voyager.minedojo.org>
