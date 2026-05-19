---
title: "UI-TARS-2 Technical Report: Advancing GUI Agent with Multi-Turn RL"
type: source
created: 2026-05-17
updated: 2026-05-17
sources:
  - ui-tars-2-wang-2025.md
arxiv_id: "2509.02544"
authors:
  - ByteDance Seed (alphabetical; corresponding: Yujia Qin, Shiguang Sg)
year: 2025
introduces:
  - "[[ui-tars-2]]"
  - "[[computer-use-agent]]"
tags:
  - 2025
status: complete
importance: high
---

# UI-TARS-2 Technical Report: Advancing GUI Agent with Multi-Turn Reinforcement Learning

[arXiv:2509.02544](https://arxiv.org/abs/2509.02544) — ByteDance Seed, September 2025.

Authors are listed alphabetically in the paper; correspondence is to **Yujia Qin** and **Shiguang Sg** (`yujia.qin@bytedance.com`, `shiguang.sg@bytedance.com`). The note on the local filename "wang-2025" reflects an early ingest convention, not the actual first author — the paper has no single first author.

## Summary

UI-TARS-2 is an **end-to-end native GUI agent** from [[bytedance-seed]] that unifies perception, reasoning, action, and memory inside a single policy, and is trained with a **stabilized multi-turn reinforcement learning** pipeline rather than a modular planner+executor stack. It is the direct successor to UI-TARS-1 / UI-TARS-1.5 ([[ui-tars-1]], arXiv:2501.12326), and is positioned as the strongest open computer-use agent at release time, outperforming Anthropic's Claude Computer Use and OpenAI's CUA on several head-to-head benchmarks.

The methodology rests on four pillars: (1) a **data flywheel** that co-evolves model and corpus through continual pre-training, SFT, rejection sampling, and RL, producing a steady stream of long-horizon trajectories that would otherwise be prohibitively expensive to collect; (2) a **stabilized multi-turn RL framework** built on PPO with Decoupled-GAE (from VAPO / VC-PPO), reward shaping, value pretraining, asynchronous server-mode rollouts, and streaming updates from a partially-filled rollout pool to absorb long-tail trajectories; (3) a **hybrid GUI-centered environment** where pixel-level mouse/keyboard actions interoperate with file-system access and a terminal, broadening the task space beyond pure click-and-type; and (4) a **unified sandbox platform** orchestrating cloud VMs, browsers, mobile, and game sandboxes behind a single API for million-scale reproducible rollouts.

Empirically, UI-TARS-2 reaches **88.2 on Online-Mind2Web**, **47.5 on OSWorld**, **50.6 on WindowsAgentArena**, and **73.3 on AndroidWorld**, clearly exceeding UI-TARS-1.5 and beating Claude / OpenAI agent baselines on several of these. In a 15-game suite it attains a mean normalized score of **59.8** (~60% of human), surpassing OpenAI CUA by ~2.4× and Claude Computer Use by ~2.8×, and remains competitive with OpenAI o3 on LMGame-Bench. With its GUI-SDK extension (terminal + external tools), the model also generalizes to long-horizon information-seeking (BrowseComp) and software-engineering benchmarks.

## Key Points

- **Native end-to-end policy.** Perception, reasoning, action, and memory are unified in one model — no separate planner, grounder, or memory module. This is the "native agent model" line that [[ui-tars-1]] argued for, scaled up.
- **Data flywheel.** Continual pre-training (in-situ annotated GUI traces) → SFT (interactive annotation) → rejection sampling → multi-turn RL, with each stage feeding cleaner trajectories back into the next. Designed specifically for the data-scarcity problem of long-horizon GUI traces.
- **Stabilized multi-turn RL.** PPO with reward shaping (mostly outcome-based, with format rewards / length penalties to prevent premature termination or infinite loops), **Decoupled-GAE** with separate λ for policy and critic to combat value decay over long sequences, value pretraining, and adaptive advantage estimation.
- **Asynchronous server-mode rollouts + streaming training.** Rollouts run in a dynamic pool; training kicks off once enough completed traces accumulate, while long-tail incomplete traces stay in the pool. Conceptually similar to Kimi-Researcher's async rollout design.
- **Stateful environments.** Tool invocations preserve execution state across turns so multi-step workflows can carry persistent context (file edits, terminal sessions, browser tabs).
- **Hybrid GUI / file / terminal env.** Action space is not GUI-only: agent can read/write files and shell out, which is essential for SWE-bench-style and information-seeking tasks. Mirrors the broader industry shift from "screen-only" CUAs toward computer-using agents.
- **All-in-One GUI Sandbox.** Heterogeneous environments (desktop VMs, mobile, browser, game sandboxes) under a consistent API; engineered for reproducibility and millions of rollouts.
- **Vertical agent merging.** Domain-specialized variants (e.g., game-tuned, GUI-tuned) are merged via parameter interpolation rather than maintaining separate checkpoints — interesting practical engineering note.
- **Benchmark headline numbers**: Online-Mind2Web **88.2**, OSWorld **47.5**, WindowsAgentArena **50.6**, AndroidWorld **73.3**, 15-game suite mean normalized **59.8**.
- **Positioning.** At publication, the strongest open / non-Anthropic [[computer-use-agent]], and the clearest public account of what large-scale multi-turn agentic RL actually looks like in production.

## Entities

- [[bytedance]] — parent org.
- [[bytedance-seed]] — research lab that produced UI-TARS-2.

## Concepts

- [[computer-use-agent]] — the broader category UI-TARS-2 belongs to.
- [[gui-agent]] — narrower framing focused on screen-driven control.
- [[agentic-rl]] — RL applied to long-horizon tool-using agents; UI-TARS-2 is a flagship example.
- [[multi-turn-rl]] — the specific training paradigm; the paper's core technical contribution is making this stable at scale.
- [[osworld]] — primary GUI agent benchmark; UI-TARS-2 scores 47.5.

## Quotes

> UI-TARS-2, a native GUI-centered agent model that addresses these challenges through a systematic training methodology: a data flywheel for scalable data generation, a stabilized multi-turn RL framework, a hybrid GUI environment that integrates file systems and terminals, and a unified sandbox platform for large-scale rollouts.

> On GUI benchmarks, it reaches 88.2 on Online-Mind2Web, 47.5 on OSWorld, 50.6 on WindowsAgentArena, and 73.3 on AndroidWorld, outperforming strong baselines such as Claude and OpenAI agents.

> Following VAPO and VC-PPO, UI-TARS-2 integrates several critical enhancements to broaden the exploration space and improve stability, especially in long-horizon settings.

## References

- Paper: [arXiv:2509.02544](https://arxiv.org/abs/2509.02544)
- Code / project: https://github.com/bytedance/ui-tars, https://github.com/bytedance/UI-TARS-desktop
- Demo: https://seed-tars.com/showcase/ui-tars-2
- Predecessor: [[ui-tars-1]] (arXiv:2501.12326)
- Related RL ingredients: VAPO, VC-PPO, Kimi-Researcher (async rollouts)
- Benchmarks cited: Online-Mind2Web, [[osworld]], WindowsAgentArena, AndroidWorld, LMGame-Bench, BrowseComp
