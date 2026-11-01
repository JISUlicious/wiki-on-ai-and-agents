---
title: "Agensh: Scaling Organizational Intelligence to 1,024 Agents"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - agensh-scaling-organizational-intelligence-1024-agents-zhan-2026.md
arxiv_id: "2609.26781"
authors:
  - Zhihao Zhan
  - Ting Song
  - Li Dong
  - Shaohan Huang
  - Jianxun Lian
  - Yan Xia
  - Furu Wei
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Agensh: Scaling Organizational Intelligence to 1,024 Agents

**Zhihao Zhan, Ting Song, Li Dong, Shaohan Huang, Jianxun Lian, Yan Xia, Furu Wei** ([[microsoft-research|Microsoft Research]]). [arXiv:2609.26781](https://arxiv.org/abs/2609.26781), v1, cs.CL, 22 Sep 2026. Technical report; code at github.com/microsoft/Agensh.

## Summary

Existing multi-agent harnesses (Claude Code agent teams, Codex sub-agents, Copilot fleet, Kimi Agent Swarm) use an **orchestrator–worker** structure, so the lead agent's capacity to plan and integrate caps how far the system scales. Agensh removes the orchestrator. Every worker gets the **same prompt except its ID** and runs a five-step loop: gather context, claim a sub-task, act, verify, merge. Workers coordinate through three pieces of infrastructure:
- a **shared workspace** (a Gitea repository with branches, pull requests and issues);
- a **message interface** (Mattermost, with a task channel and direct messages);
- **shared context**, adapted from DeLM: an append-only log of typed entries (OBSERVED, FACT, FAIL, CLAIM, PATCH_SUMMARY) that workers can search with a context-grep tool.

The loop is implemented as prompt instructions, not runtime code, so Agensh can sit on top of any single-agent harness through a thin adapter. The experiments use Copilot.

## Key points

- **Five hardest ProgramBench tasks** (FFmpeg, gromacs, pandoc, PHP-src, ctags; 6 h budget, no Internet; GPT-5.6-sol high). Mean final test-pass rate is **19.31% with 1 agent → 20.68% (8) → 26.52% (32) → 28.78% (128)**: +9.47 pp, about 49% relative.
- **pandoc to 1,024 agents.** Pass rate goes **33.89% (1) → 50.94% (128) → 55.06% (1,024)**. The last 8× in agents adds only 4.12 pp.
- **Latency.** On pandoc, 128 agents pass 30% at the 30-minute checkpoint. 32 agents get there at 60 minutes and 8 agents at 90 minutes; 1 agent stays below 30% for the first 2 h.
- **Qualitative emergence by scale.** At 8 agents, workers agree on interfaces and resolve overlapping claims. At 32, a worker withdraws a peer approval after finding a counterexample. At 128, workers pick reviewers by expertise and settle on a standard integration protocol. At 1,024, several workers act as integrators and specialists take over failed attempts.

## What it does not show

- **One run per configuration.** The paper reports no seeds, variance or confidence intervals. The 1,024-agent result is a single pandoc run.
- **No cost or token accounting.** 1,024 GPT-5.6-sol agents running for 6 h is a very large compute bill, and there is no matched-compute single-agent baseline (such as a continued or restarted single agent with 1,024× the tokens).
- **No orchestrator baseline.** The paper says orchestrators are the bottleneck but never tests one. Claude Code teams, Kimi Swarm and similar systems are not run on these tasks.
- **Returns diminish steeply.** On pandoc, going from 128 to 1,024 agents (8×) adds 4.12 pp. "Agent count as a new scaling dimension" rests on 4 to 5 points per curve.
- The emergent-cooperation evidence is a set of anecdotes chosen from trajectories, not measured frequencies.

## Concepts & entities

- [[multi-agent]] · [[multi-agent-memory]]: orchestrator-free coordination through shared state. Compare [[gitswarm-decentralized-compounding-inference-shah-2026|GitSwarm]] and [[agora-git-shared-memory-collective-autoresearch-zhang-2026|Agora]], which also use Git as the shared substrate. Agensh adds a chat channel and a typed findings log.
- [[subagents]] · [[learned-orchestration]]: the orchestrator–worker pattern Agensh positions itself against. [[kimi-k3-moonshot-2026]] covers the Kimi Agent Swarm line.
- [[agent-benchmark]]: ProgramBench (program reconstruction from an execute-only reference). See also [[harness-of-harness-multi-day-autonomous-development-yan-2026]].
- [[inference-time-compute]]: agent count used as the scaling axis.
- [[microsoft-research]] · [[claude-code]]
- [[programbench]]: benchmark page; scores across papers use different subsets and metrics.

## References

- [arXiv:2609.26781](https://arxiv.org/abs/2609.26781)
