---
title: "MIMESIS: Learning User Simulators as Training Environments for Interactive Agents"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - mimesis-user-simulators-training-environments-phan-2026.md
arxiv_id: "2610.09484"
authors:
  - Hoang Phan
  - Dat Huynh
  - Andrey Zhmoginov
  - Qi Zeng
  - Wancen Mu
  - Yue Cao
  - Shengjie Bi
  - Yun He
  - Changdae Oh
  - Deren Lei
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# MIMESIS: Learning User Simulators as Training Environments for Interactive Agents

**Hoang Phan, Dat Huynh, Andrey Zhmoginov, Qi Zeng, Wancen Mu, Yue Cao, Shengjie Bi, Yun He, Changdae Oh, Deren Lei** (Meta Superintelligence Labs, [[meta]]; NYU; UW–Madison): [arXiv:2610.09484](https://arxiv.org/abs/2610.09484), v2, cs.AI, 8 Oct 2026. Preprint; no venue. It shares authors with [[sharpening-tax-in-post-training-oh-2026]].

## Summary

Interactive agents are trained and evaluated against *simulated users*, usually an off-the-shelf assistant LLM prompted to role-play. The authors show that such users are **too cooperative**. On [[tau-bench|τ-bench]], a fixed GPT-5.5 agent succeeds 63.6% of the time with real users. Swap in GPT-5.5, Gemini-3.8-Flash or Claude-Opus-5 as the user and success rises to **82.4–84.4%**. Pretrained Qwen models as users push it the other way, down 15–18 points. MIMESIS, a purpose-built simulator, lands at 65.5% (4B) and 66.1% (9B).

MIMESIS is built on Qwen3.5-4B/9B. Training has three parts: mid-training on human–assistant conversations from the user's side, explicit (private) reasoning supervision, and **13 behavioral patterns** mined from real user interactions (for example, gradually shifting the goalposts). The frozen simulator then serves as the RL environment for agents. The paper also proposes **Coached On-Policy Self-Distillation (CSD)**. A coach turns the simulator's private reasoning and next utterances into short coaching notes. A coached copy of the agent acts as a teacher and gives dense token-level targets. At deployment the agent sees only the public dialogue.

## Key points

- **Simulator fidelity.** Four simulation benchmarks:
  - SOUL-Index: MIMESIS-9B **65.7**, vs Claude-Opus-5 64.9 and GPT-5.5 64.1. The margin is small. Its largest lead is the conversational axis: 75.1 vs 61.7 for Ditto-8B.
  - RealUserSim PT3 Fidelity Index: **94.0 vs 80.6** for Claude-Opus-5, a 13.4-point gap. Most of it comes from information flow (91.3 vs 69.5) and pacing (91.2 vs 72.5). Persona scores are near ceiling for every model.
  - SimulatorArena: Turing distance is 3.6 lower than Claude-Opus-5's.
  - GPT-5.5 and Claude keep the top scores on the SOUL SS and ROLE axes.
- **Agent training.** Setup follows UserRL: multi-turn [[grpo|GRPO]] over 8 Gym environments, 3 of them held out, and evaluation under **9 user models never seen in training**.
  - Swapping GPT-5.5 for MIMESIS-9B as the training user raises the mean score from **26.10 to 29.54**. The gain holds under every evaluation user, from +1.46 under Claude-Opus-5 to +4.55 under Gemini-3.8-Flash.
  - **CSD** on top of that: 29.54 → **31.09**, again better under all 9 users.

## What it does not show

> [!warning] Newsletter vs paper
> "+13.4 over Claude Opus 5 fidelity" is right, but only for RealUserSim's Fidelity Index. On SOUL, the overall lead over Claude is 0.8 points. Read the "13.4" as one benchmark's result.

- **Agents are never evaluated with real humans.** All nine evaluation users are simulators (frontier models plus released simulators). Generalization to unseen *simulators* is shown, but not generalization to people.
- Absolute agent scores are low (around 26–31 mean), and gains are 1.5–4.6 points per user.
- Making users harder could teach agents to be over-cautious. Calibration of agent clarification behavior is not studied.

## Concepts & entities

- [[tau-bench]] · [[agent-evaluation]]: simulated users set the measured difficulty. Frontier "users" inflate success by about 20 points.
- [[agentic-rl]] · [[grpo]] · [[on-policy-distillation]]: CSD is a coached-teacher variant of on-policy self-distillation.
- [[sycophancy]]: the over-cooperative user is the mirror image of the over-agreeable assistant.
- [[multi-user-llm-agents-yang-2026]] · [[salesforce-koa-enterprise-agentic-tool-use-salesforce-2026]] · [[gauge-when-not-to-trust-llm-as-a-judge-bodhwani-2026]]: other pages that touch user simulation.
- Same batch: [[critical-state-rl-trainable-states-tool-use-chen-2026]] (multi-turn credit), [[sharpening-tax-in-post-training-oh-2026]] (same lab).
- [[meta]] · [[qwen]].

## References

- [arXiv:2610.09484](https://arxiv.org/abs/2610.09484) · Code: VietHoang1512/mimesis
