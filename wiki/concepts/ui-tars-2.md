---
title: UI-TARS-2
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - ui-tars-2-wang-2025.md
status: complete
importance: high
tags:
  - 2025
---

# UI-TARS-2

**UI-TARS-2** is [[bytedance]] Seed's open-source [[computer-use-agent]], a native end-to-end GUI agent that unifies perception, reasoning, action, and memory in one policy and is trained with stabilized multi-turn reinforcement learning. At release (September 2025) it was the strongest open / non-Anthropic computer-use agent, beating Claude Computer Use and OpenAI CUA on several head-to-head benchmarks.

## Core mechanism

- **Data flywheel** — continual pre-training on in-situ annotated GUI traces → SFT (interactive annotation) → rejection sampling → multi-turn RL, with each stage feeding cleaner trajectories back into the next. Engineered specifically for the data-scarcity problem of long-horizon GUI traces.
- **Stabilized multi-turn RL** — PPO with **Decoupled-GAE** (separate λ for policy and critic, from VAPO / VC-PPO) to combat value decay over long sequences, plus value pretraining, reward shaping (mostly outcome-based with format rewards and length penalties), and adaptive advantage estimation. Asynchronous server-mode rollouts feed a streaming training loop that absorbs long-tail incomplete trajectories.
- **Hybrid GUI / file / terminal environment** — action space extends beyond pixel-level mouse/keyboard to include file-system access and shell commands, enabling SWE-bench-style and information-seeking workflows. Tool invocations preserve execution state across turns.
- **All-in-One GUI Sandbox** — desktop VMs, browsers, mobile, and game sandboxes orchestrated behind a single API for million-scale reproducible rollouts.
- **Benchmark numbers** — [[osworld]] **47.5**, Online-Mind2Web **88.2**, WindowsAgentArena 50.6, AndroidWorld 73.3, 15-game suite mean normalized 59.8 (~60% of human, ~2.4× OpenAI CUA, ~2.8× Claude Computer Use). Vertical specialists are merged via parameter interpolation rather than maintained as separate checkpoints.

## What it advances

UI-TARS-2 supersedes UI-TARS-1 (arXiv:2501.12326) and is the clearest public account of what large-scale [[multi-turn-rl]] actually looks like in production for [[computer-use-agent]]s. It demonstrates that the "native agent model" thesis — unified perception/reasoning/action/memory in one policy — scales, and that the agentic-RL stabilization tricks (Decoupled-GAE, value pretraining, async rollouts) are necessary at long horizons.

## Relation to other concepts

- Instance of the [[computer-use-agent]] / [[gui-agent]] category; direct successor to [[ui-tars-1]].
- Flagship example of [[agentic-rl]] applied at production scale.
- Trains on and benchmarks against [[osworld]], Online-Mind2Web, WindowsAgentArena, AndroidWorld.
- Async rollout design is conceptually similar to Kimi-Researcher.
- Contrasts with cross-domain [[magma]] (pretraining-heavy, GUI + robotics) — UI-TARS-2 is GUI-specialist and RL-heavy.
- Produced by [[bytedance]] / ByteDance Seed.

## References

- [[ui-tars-2-wang-2025]] — Technical report; [arXiv:2509.02544](https://arxiv.org/abs/2509.02544)
- Code: <https://github.com/bytedance/ui-tars>, <https://github.com/bytedance/UI-TARS-desktop>
