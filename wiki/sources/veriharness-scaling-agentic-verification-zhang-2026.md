---
title: "VeriHarness: Scaling Agentic Verification for Long-Horizon Tasks"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - veriharness-scaling-agentic-verification-zhang-2026.md
arxiv_id: "2610.00972"
authors:
  - Caiqi Zhang
  - Rujun Han
  - Zifeng Wang
  - Zoey CuiZhu
  - Nigel Collier
  - Tomas Pfister
  - Chen-Yu Lee
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# VeriHarness: Scaling Agentic Verification for Long-Horizon Tasks

**Caiqi Zhang, Rujun Han, Zifeng Wang, Zoey CuiZhu, Nigel Collier, Tomas Pfister, Chen-Yu Lee** (University of Cambridge; [[google|Google Cloud AI Research]]) — [arXiv:2610.00972](https://arxiv.org/abs/2610.00972), v1, cs.AI, 1 Oct 2026. Preprint. The work was done during Zhang's Google internship. Site: veriharness.com.

## Summary

This is test-time verification **without a stronger judge**. The generator and verifier are the same model, and the verifier never sees reference answers or rubrics.

The starting observation comes from a case study of N = 10 rollouts. **Consensus can hide errors**: about a third of agreed values were wrong. **Disagreement often contains the correct answer**, even when the majority is wrong. So the verifier gets a harness with a workspace, evidence tools and reusable verification skills. It runs two separate investigations:

- A **disagreement resolver** checks the competing claims against source files, data and constraints.
- A **consensus challenger** tries to refute the shared claims and looks for omitted requirements.

Adjudication then selects a base artifact and applies an evidence-backed revision plan.

## Key points

- **Five workspace benchmarks**: APEX-Agents, Workspace-Bench Lite, WorkBuddy Bench, SpreadsheetBench 2 and JobBench. Two models act as both generator and verifier: Gemini 3.5 Flash and Claude Opus 4.8. Each task has 10 frozen rollouts, and each verifier is run 3 times.
- **Selection alone** improves on the single rollout by **+4.4 (Flash) and +4.1 (Opus)** on average. It beats every baseline in all 10 model–benchmark cells. The baselines are majority vote, best-of-N judging, a pairwise tournament, LLM-as-a-Verifier, and an agentic verifier with the same tools but no protocol.
- **Tools alone are not enough.** The agentic verifier with environment access recovers only about half of the harness's gain.
- **With revision**, the gain over a single rollout rises to **+6.2 (Flash) and +6.4 (Opus)**. Plain pool aggregation without evidence checks stays below this on every benchmark.
- **Gains come from disagreement.** On APEX-Agents with Opus, consensus pools (154) gain only +0.6 over the pool mean, while disputed pools (262) gain +6.1.
- **The protocol works inside other harnesses.** Run in Gemini CLI, Claude Code or Codex, it keeps most of the gain with Flash. With Opus the CLIs fall back to single-rollout level on JobBench.
- **Skills self-improve.** Of the 95 checks in the evolved skill libraries, 42 are new and 48 make a human-written check concrete.
- **Dataset release**: about 26,000 rollouts, which cost over $100k to produce.

> [!warning] Newsletter vs paper
> The figures are correct, and "Google" is half the affiliation; the first author is at Cambridge. The **+6.2 / +6.4** are measured against a **single rollout**, while every method draws on a pool of **10** rollouts. Generation cost is equal across verifiers but 10× the baseline the gain is quoted against. Selection alone gives +4.4 / +4.1.

## What it does not show

- **No per-rollout score.** The harness gives no calibrated scalar per rollout, so it cannot yet serve as an RL reward.
- **Same-model only.** Stronger-verifier settings are out of scope.
- **Latency.** Verification is a multi-turn investigation that adds wall-clock time.
- **Blind spot.** About 70% of the consensus errors left standing were cases where the challenger checked a correct intermediate step and missed the later wrong one.

## Concepts & entities

- [[verifier]] · [[llm-as-a-judge]] · [[self-consistency]] · [[test-time-compute]] — evidence-checking verifiers vs voting and judging.
- [[agent-as-a-judge]] · [[agent-evaluation]] — agentic verification of long-horizon artifacts.
- [[agentic-harness-engineering]] · [[agent-skills]] · [[skill-optimization]] — a verifier harness whose skills evolve.
- [[harness2-recursive-agent-harnessing-xu-2026]] — the same Google Cloud AI group; also works with no ground truth.
- [[claude-code]] · [[claude-opus-4-8]]

## References

- [arXiv:2610.00972](https://arxiv.org/abs/2610.00972)
