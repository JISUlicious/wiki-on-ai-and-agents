---
title: RL Environment Synthesis
type: concept
created: 2026-10-29
updated: 2026-10-29
sources:
  - autogym-blueprint-first-verifiable-agent-gyms-noronha-2026.md
  - vera-verifiable-environments-agentic-co-evolution-liu-2026.md
  - skillgym-internalizing-human-skills-ge-2026.md
  - autoenvscaling-data-flywheel-terminal-agents-yu-2026.md
  - agent-world-real-world-environment-synthesis-dong-2026.md
  - skill2env-capability-oriented-environment-synthesis-allspark-2026.md
  - envharness-awakening-static-worlds-huang-2026.md
  - from-trainee-to-trainer-chen-2026.md
  - frognano-training-a-4b-coding-agent-via-online-task-synthesis-kim-2026.md
status: draft
importance: high
tags:
  - 2026
  - agent
---

# RL Environment Synthesis

**RL environment synthesis** is the automatic generation of training environments for agents: a task, an executable environment (often a Docker sandbox), and a [[verifier]] or rubric that scores the outcome. The motivating claim across these papers is that for [[agentic-rl]], hand-built environments, not algorithms, are the scarce input. Hand-built suites are "blind to an agent's weaknesses, and quickly left behind as it improves" ([[envharness-awakening-static-worlds-huang-2026]]), and fixed task sets invite [[benchmark-contamination]].

## Design axes

| Axis | Options seen in the sources |
|---|---|
| **Seed** | Domain seed or prior trajectories (AutoGym); benchmark trajectories (VERA); human skill folders (SkillGym, Skill2Env); repository snapshots (FrogNano); themes and MCP tool ecosystems (Agent-World); an existing static environment (EnvHarness) |
| **Generation order** | Task first, then check solvability, vs **blueprint first** (AutoGym, Skill2Env's "task blueprint") so solvability is built in |
| **Verification** | Code checkers and reference solutions; binary rubrics plus an agent judge (VERA); human-built verifier left untouched (EnvHarness) |
| **Difficulty control** | Explicit generation parameters (AutoGym); a target pass-rate band for the current solver (AutoEnvScaling, FrogNano); iterative hardening (Skill2Env) |
| **Who generates** | A fixed frontier model (AutoGym uses Claude Sonnet 4.6) vs **the policy itself** (From Trainee to Trainer, EnvHarness, AutoEnvScaling's self-play arm) |

## Systems

- [[autogym-blueprint-first-verifiable-agent-gyms-noronha-2026|AutoGym]]: blueprint-first gyms with a failure-driven curriculum; repair dropped C2's hard share from 57% to 39%, showing many "hard" tasks were broken. Its main evidence is difficulty and model separation, not training gains.
- [[vera-verifiable-environments-agentic-co-evolution-liu-2026|VERA]]: 9,000+ stage-wise restartable sandboxes and model–harness co-evolution; each axis supplies about half the gain in its ablation.
- [[skillgym-internalizing-human-skills-ge-2026|SkillGym]]: 2,756 skill-derived environments; every reported gain is from SFT on teacher trajectories (Terminal-Bench 2.1 +19.10), no RL run.
- [[skill2env-capability-oriented-environment-synthesis-allspark-2026|Skill2Env]]: 2,963 tasks from skill folders via 100 difficulty patterns; SFT raises a seven-benchmark average 36.6 → 45.0.
- [[autoenvscaling-data-flywheel-terminal-agents-yu-2026|AutoEnvScaling]]: an agentic proposer writes Harbor environments calibrated to the solver; beats a matched static pool (Tmax) by +3.8 to +7.4 on held-out Terminal-Bench 2.1.
- [[agent-world-real-world-environment-synthesis-dong-2026|Agent-World]]: environment-task discovery plus a self-evolving arena; reports gains over environment-scaling baselines across 23 benchmarks.
- [[envharness-awakening-static-worlds-huang-2026|EnvHarness]]: reshapes static environments through `reset`/`step` (Stage, Contract, Chain) instead of generating new ones; up to +6.5 in online GRPO.
- [[from-trainee-to-trainer-chen-2026|From Trainee to Trainer]]: the current RL checkpoint redesigns its next-stage environment and is a better environment engineer than the base model.
- [[frognano-training-a-4b-coding-agent-via-online-task-synthesis-kim-2026|FrogNano]]: online task synthesis (TaskPilot) rewrites tasks the checkpoint always or never solves; five synthetic rounds beat a filtered real-task baseline by about 8 points on SWE-bench Verified.

## Cautions

- **"Gym" does not mean RL results.** AutoGym's only training evidence is one 8B GRPO reward curve; SkillGym used SFT only. Both source pages flag the newsletter framing.
- **Validation is not correctness.** AutoEnvScaling notes that "reference passes, empty solution fails" does not prove an environment is right.
- **Same-model coupling.** VERA uses one model for judging, verification and attribution; AutoEnvScaling's self-play starts from frontier-distilled data.
- **Scale is rarely varied.** VERA never varies environment count, so "scaling environments" is not itself measured.

## Related

- [[agentic-rl]] · [[harnessed-agentic-rl]] · [[verifier]] · [[agent-skills]] · [[swe-gym]] · [[recursive-self-improvement]] · [[self-improving-agent]]
