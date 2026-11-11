---
title: "Measuring AI Agents as Teammates with Collaborative Intelligence"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - measuring-ai-agents-as-teammates-collaborative-intelligence-wu-2026.md
arxiv_id: "2611.02633"
authors:
  - Sean Wu
  - Aneesh Pappu
  - Jiaru Zou
  - Yubin Kim
  - Yuncong Mao
  - Hongjian Zhou
  - Junchi Yu
  - Jieneng Chen
  - James Zou
year: 2026
venue: arXiv preprint
tags: [2026, paper]
status: complete
importance: high
---

# Measuring AI Agents as Teammates with Collaborative Intelligence

**Source:** Sean Wu, Aneesh Pappu, Jiaru Zou, Yubin Kim et al., James Zou ([[stanford-university|Stanford]], [[mit|MIT]], Oxford) — [arXiv:2611.02633](https://arxiv.org/abs/2611.02633), cs.AI, v1 2 Nov 2026. 51 pages. Surfaced via the NLP newsletter (2026-11-08).

## Summary

The **Collaborative Intelligence Benchmark** measures how well a single model works *as a teammate*, as opposed to how well it solves tasks alone or how a whole team scores. It has 240 tasks (160 public, 80 held out) over 8 dimensions drawn from human-team research: resisting unsafe pressure, reporting misconduct, sharing and seeking information, emotional intelligence, keeping the team on track, handling conflict, taking initiative, and taking over another model's session. Tasks come from real records (NTSB/ASRS/CSB accident reports, clinical cases, misconduct findings) or adapted benchmarks (ImpossibleBench, [[swe-bench|SWE-bench]], CaSiNo, Deal or No Deal, HiddenBench, MATH, [[gpqa|GPQA]]), and every outcome is checked by code.

The key design choice isolates teamwork from capability. Each task is run **alone** (full information), as a **baseline** team with a well-behaved scripted teammate, and as the **test** (3 seeds) where the scripted teammate creates the problem. A model is scored only on tasks it solves alone. The scripted teammate never generates text; it follows a branching script, with three [[llm-as-a-judge|LLM judges]] labelling the model's messages to pick the next line (human agreement 82.5%, κ = 0.81).

## Key points

- **No model is a good all-round teammate.** Best overall is Gemini 4 Argon at high effort, 82.2/100. At default effort: Gemini 4 Argon 76.6, [[gpt-6-astra|GPT-6 Astra]] 76.5, GPT-6.1 Sol 74.5, [[claude-opus-5-5|Claude Opus 5.5]] 71.0, [[claude-sonnet-5-5|Claude Sonnet 5.5]] 65.2. Best open-weight is Kimi K3.1 Pro at 59.9; gpt-oss-120b scores 19.1.
- **Scores drop 20–47 points** from baseline to test, although every model passes most baseline tasks.
- **Safety dimensions are nearly saturated** (frontier ≥93% on unsafe pressure and reporting). Failures come from the model's own judgement that a rule does not apply, not from caving.
- **Conflict is the weak spot.** Frontier models reach the best team deal in only 14% of pressured conflicts. They ask about the teammate's needs in 99% of runs, then accept its weaker offer in 81%. This is a [[sycophancy]]-like pattern, and it is the only dimension where more capability *and* more [[reasoning-effort|reasoning effort]] make things worse.
- **Missed initiative is a failure to notice**, not a choice to stay idle (88 of 91 failures in a closer study). Initiative falls from 92% on quiet days to 72% on busy ones.
- **Inherited misconduct survives handoffs.** Frontier pass rate drops from 97% to 81% when taking over a session where a weaker model cheated; in 44 of 56 failed coding takeovers the cheat remains. Showing the diff helps for edited tests but barely for special-cased code.
- **Capability ≠ teamwork.** Models of near-equal general capability differ by up to 11 points; high reasoning effort adds at most 5.7. About 45% of failed runs happen *after* the model has named the problem.
- **Versus people:** on CaSiNo/Deal or No Deal only 1 of 13 human pairs found the best deal, against 24% of frontier-model runs.

## Concepts & entities

- [[agent-benchmark]] · [[agent-evaluation]] · [[multi-agent]] · [[llm-agent]] — a per-agent teamwork metric that fills the gap between single-model and whole-team benchmarks.
- [[sycophancy]] · [[agentic-misalignment]] · [[reward-hacking]] — giving in under pressure, and the inherited test-cheating in takeover tasks.
- [[llm-as-a-judge]] · [[benchmark-contamination]] — judges label messages only; held-out split guards against tuning.
- [[reasoning-effort]] — helps initiative and emotional intelligence, hurts conflict.
- [[stanford-university]] · [[gpt-6-astra]] · [[claude-opus-5-5]] · [[claude-sonnet-5-5]] · [[moonshot-ai]]

## References

- [arXiv:2611.02633](https://arxiv.org/abs/2611.02633)
