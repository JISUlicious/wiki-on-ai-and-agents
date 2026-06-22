---
title: Computer-Use Agent
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - ui-tars-2-wang-2025.md
  - magma-yang-2025.md
status: complete
importance: high
tags:
  - 2025
---

A **computer-use agent (CUA)** is a vision-language-model agent that drives a screen — clicking, scrolling, typing, navigating menus, and increasingly running terminals and editing files — to accomplish tasks across GUIs, browsers, and operating systems. The screen is the action surface: the agent perceives pixels (and sometimes accessibility/DOM/view-hierarchy structure), reasons over the visual state, and emits low-level UI actions. The pattern emerged in 2024-2025 with Anthropic Computer Use, OpenAI CUA, and [[ui-tars-1]], and reached production maturity in 2025 with [[ui-tars-2]] (end-to-end native GUI agent trained with multi-turn RL) and [[magma]] (unified VLA covering both GUI and robotics from a single backbone). CUAs are increasingly the front-end for long-horizon work that mixes information-seeking, software operation, and now physical manipulation.

## Core mechanism

- **Screen as action surface**: action space is pixel-level mouse and keyboard primitives (click(x,y), drag, scroll, type, hotkey), often interleaved with structured tool calls (file read/write, shell exec) for hybrid environments. The agent observes screenshots (plus sometimes DOM/view-hierarchy) and emits the next action.
- **Two architectural flavors**: (1) pure VLM grounding — Anthropic Computer Use, [[ui-tars-2]] — where one model handles perception, reasoning, action, and memory in a single native policy; vs. (2) unified VLA — [[magma]] — where the same backbone covers GUI clicks and 7-DoF robot actions via a shared output format ([[set-of-mark]] for action grounding, [[trace-of-mark]] for action planning).
- **Multi-turn agentic RL**: the load-bearing training paradigm. [[ui-tars-2]] uses PPO with Decoupled-GAE (from VAPO/VC-PPO), reward shaping, value pretraining, and asynchronous server-mode rollouts to keep long-horizon (sometimes hundreds of turns) RL stable; outcome rewards plus light format/length penalties.
- **Data flywheel**: continual pre-training on in-situ annotated GUI traces → SFT on interactive annotation → rejection sampling → multi-turn RL; each stage feeds cleaner trajectories back into the next, addressing the long-horizon-trace data-scarcity problem.
- **Hybrid GUI + file + terminal environments**: modern CUAs are no longer screen-only — they shell out, edit files, and persist execution state across turns, which is essential for SWE-style and information-seeking tasks. Mirrors the industry shift from pure click-and-type CUAs to general computer-using agents.

## What it advances

Computer-use agents unify the previously fragmented spaces of browser automation, RPA, GUI testing, and embodied robotics under a single VLM-driven control paradigm. They turn arbitrary software (and increasingly arbitrary physical robots) into addressable action surfaces, dramatically widening the set of tasks an LLM agent can attempt without bespoke API integration. The 2025 generation ([[ui-tars-2]], [[magma]], Anthropic CU) is the first to credibly compete with humans on benchmarks like OSWorld, WindowsAgentArena, AndroidWorld, and Online-Mind2Web, and to demonstrate cross-domain (digital + physical) generalization from a single set of weights.

## Relation to other concepts

Narrower subtype: [[gui-agent]] (screen-only, no physical action). Broader unification: [[vision-language-action-model]] (Magma's framing — one model for GUI + robotics). Grounding technique: [[set-of-mark]] (numerical overlays on clickable elements) and trace-of-mark (future trajectory of marked points). Training paradigm: [[agentic-rl]] with multi-turn rollouts. Compare to API-using [[llm-agent]] patterns that operate over structured tool calls rather than pixels. Evaluation suites: [[osworld]], Online-Mind2Web, WindowsAgentArena, AndroidWorld, ScreenSpot.

## References

- [[ui-tars-2-wang-2025]] — native GUI agent with stabilized multi-turn RL; OSWorld 47.5, Online-Mind2Web 88.2.
- [[magma-yang-2025]] — unified VLA covering GUI and robotics from one backbone; CVPR 2025.
- Related: Anthropic Claude Computer Use, OpenAI CUA, [[ui-tars-1]] (arXiv:2501.12326).

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[preact-computer-using-agents-li-2026]] — PreAct: compile CUA runs into replayable state machines
