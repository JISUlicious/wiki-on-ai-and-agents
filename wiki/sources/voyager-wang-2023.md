---
title: "Voyager: An Open-Ended Embodied Agent with Large Language Models"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - voyager-wang-2023.md
arxiv_id: "2305.16291"
authors:
  - Guanzhi Wang
  - Yuqi Xie
  - Yunfan Jiang
  - Ajay Mandlekar
  - Chaowei Xiao
  - Yuke Zhu
  - Linxi Fan
  - "[[anima-anandkumar]]"
first_author: Guanzhi Wang
year: 2023
introduces:
  - "[[voyager]]"
  - "[[skill-library]]"
  - "[[procedural-memory]]"
tags:
  - 2023
status: complete
importance: high
---

# Voyager: An Open-Ended Embodied Agent with Large Language Models

**Source:** `sources/voyager-wang-2023.md`
**arXiv:** [arXiv:2305.16291](https://arxiv.org/abs/2305.16291)
**Project page:** [voyager.minedojo.org](https://voyager.minedojo.org)
**Authors:** Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi "Jim" Fan, [[anima-anandkumar]]
**Affiliations:** NVIDIA, Caltech, UT Austin, Stanford, UW Madison
**Year:** 2023

## Summary

Voyager is the first LLM-powered embodied lifelong learning agent in Minecraft. It continuously explores the world, acquires diverse skills, and makes novel discoveries without human intervention. The system interacts with [[gpt-4]] via black-box prompting and in-context learning, sidestepping any model fine-tuning. Its design rests on three components: (1) an **automatic curriculum** that proposes increasingly hard goals grounded in the agent's current inventory, biome, and exploration history; (2) a **skill library** that accumulates executable JavaScript (Mineflayer) code skills, each retrievable by the embedding of its description; and (3) an **iterative prompting mechanism** that refines generated code via environment feedback, execution errors, and a GPT-4 self-verification critic, looping until the task is verified complete.

The skill library is the paper's canonical contribution to agent memory systems: it functions as a *growing procedural memory* of learned behaviors stored as compositional, interpretable code. When the curriculum proposes a new task, top-k relevant skills are retrieved by embedding similarity and injected into the GPT-4 code-generation prompt, enabling complex skills (e.g., `craftIronPickaxe`) to be synthesized by composing simpler ones (e.g., `craftStick`, `smeltIronIngot`, `makeFurnace`). Because skills are added rather than overwritten, the agent's capabilities compound monotonically over time, side-stepping the catastrophic forgetting that hobbles gradient-based continual-learning approaches.

Empirically, Voyager dominates prior LLM-agent baselines ([[react]], [[reflexion]], AutoGPT) in [[minedojo]]: 3.3× more unique items obtained, 2.3× longer map traversal, and key tech-tree milestones unlocked up to 15.3× faster. It is the only method to reach the diamond tier of the Minecraft tech tree. In zero-shot transfer to a new Minecraft world, the learned skill library lets Voyager solve novel tasks (Diamond Pickaxe, Golden Sword, Lava Bucket, Compass) from scratch where baselines fail outright — and giving AutoGPT access to Voyager's skill library substantially improves its success, isolating the skill library itself as the load-bearing component.

## Key Points

- **Skill library as vector-DB procedural memory.** Each skill is a triple of `(natural-language description, embedding of description via GPT-3.5 / `text-embedding-ada-002`, executable JS program)`. New skills are appended; retrieval is by cosine similarity over the description embedding plus environment-feedback query context.
- **Code as action space.** Programs naturally express temporally extended and compositional behaviors, unlike low-level motor commands; new skills `import` previously committed skills, so the library is reusable and combinatorial.
- **Automatic curriculum via GPT-4 novelty search.** Curriculum prompt encodes agent state (inventory, equipment, biome, time, health), prior task history, and the overarching goal "discover as many diverse things as possible" — an in-context analogue of novelty search.
- **Iterative prompting with three feedback channels.** Environment feedback (`bot.chat()` messages from primitives), execution errors from the JS interpreter, and a separate GPT-4 self-verification critic that both checks success and provides a textual critique when the task fails. Loop caps at 4 rounds before requesting a new task.
- **GPT-4 backbone, GPT-3.5 for auxiliary text tasks** (self-questioning, description generation) for cost reasons. Temperatures: 0 everywhere except curriculum (0.1 for task diversity).
- **Catastrophic-forgetting mitigation.** Because skills are append-only code artifacts indexed by embedding, no parameter update can overwrite earlier capabilities — a structural property of the procedural-memory design.
- **Headline numbers.** 63 unique items in 160 prompting iterations (3.3× baselines); 15.3× / 8.5× / 6.4× faster to wooden/stone/iron tiers; only method to reach diamond tier (1/3 trials at 102 iterations).

## Entities Mentioned

- [[anima-anandkumar]] — senior author (Caltech / NVIDIA)
- NVIDIA, Caltech, UT Austin, Stanford, UW Madison — author affiliations
- [[gpt-4]] — primary reasoning backbone (gpt-4-0314)
- GPT-3.5 (gpt-3.5-turbo-0301) — auxiliary NLP / embedding-target generation
- [[minedojo]] — Minecraft simulation framework
- Mineflayer — JavaScript bot API used as the action space

## Concepts Discussed

- [[voyager]] — the agent system itself
- [[skill-library]] — vector-indexed repository of executable code skills
- [[procedural-memory]] — the cognitive-science framing of the skill library
- [[memory-management]] — broader category this work sits within
- [[llm-agent]] — agent architecture pattern
- [[embodied-agent]] — sub-pattern for grounded / world-interacting agents
- [[automatic-curriculum]] — open-ended exploration strategy
- [[iterative-prompting]] / [[self-verification]] — self-improving code-generation loop
- [[chain-of-thought-prompting|chain-of-thought]] — used inside the code-generation prompt
- [[in-context-learning]] — mechanism by which retrieved skills shape new code
- [[react]], [[reflexion]] — baseline agent designs Voyager beats

## Notable Quotes

> "VOYAGER consists of three key components: 1) an automatic curriculum that maximizes exploration, 2) an ever-growing skill library of executable code for storing and retrieving complex behaviors, and 3) a new iterative prompting mechanism that incorporates environment feedback, execution errors, and self-verification for program improvement." — Abstract

> "Complex skills can be synthesized by composing simpler programs, which compounds VOYAGER's capabilities rapidly over time and alleviates catastrophic forgetting in other continual learning methods." — §1

> "The skills developed by VOYAGER are temporally extended, interpretable, and compositional, which compounds the agent's abilities rapidly and alleviates catastrophic forgetting." — Abstract

## References

_Original source: `sources/voyager-wang-2023.md`_
Project page: <https://voyager.minedojo.org>
arXiv: <https://arxiv.org/abs/2305.16291>
