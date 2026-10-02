---
title: "Using Claude Code: Spending your effort (Shihipar 2026)"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - using-claude-code-spending-your-effort-shihipar-2026.md
authors:
  - [[thariq-shihipar]]
year: 2026
venue: "claude.dev (Anthropic developer blog)"
publisher: Anthropic
status: complete
importance: medium
tags:
  - 2026
---

# Using Claude Code: Spending your effort

**Author**: [[thariq-shihipar|Thariq Shihipar]] · **Date**: 2026-09-25 · **Publisher**: [[anthropic|Anthropic]] (claude.dev, "Engineering")
**URL**: https://claude.dev/blog/spending-your-effort/

## Summary

A practitioner's guide to the [[reasoning-effort|effort]] setting on Opus 5.5 and Fable 5.1 in [[claude-code|Claude Code]]. It draws on two kinds of evidence: the author's own toy builds, and Anthropic's internal [[terminal-bench|Terminal-Bench 3.0]] runs. The thesis is that effort mainly modulates **how much verification, edge-case testing and independent judgement** Claude applies. On this account, higher effort pays off on tasks with many hidden edge cases, such as security, hardware and code review. The post also opens with the claim that the newest models "respond to effort without breaking the prompt cache", so `/effort` can be changed mid-conversation.

## Key points

**Anthropic-reported measurements** (internal runs, 5 attempts per task; the effort-curve chart uses 70 tasks with the 4 GPU tasks excluded, while the failure breakdown counts 370 attempts; Fable 5.1 ran with production safety interventions off and security tasks without internet, so numbers "won't line up with the public leaderboard"):
- Effort curves for Opus 5.5 and Fable 5.1 rise monotonically in both pass rate and tokens consumed. Exact values are in a chart only.
- **Fable 5.1, all attempts (n=370)**: 140 passed at low vs. 214 at max. "A bug its tests missed" fell from 40 to 14. "Picked the wrong reading" *rose* from 25 to 47. The failure labels come from a model judge and are "approximate".
- **By category, Fable 5.1 low → top effort**: Security 64→87%, Hardware 34→75%, ML 54→73%, Science 41→61%, Software 43→56%, Media 18→30%, Operations 12→22%.
- **Single-task examples**:

  | Model | Task | Score change |
  |---|---|---|
  | Fable 5.1 | `html-js-filter` | 1/5 → 5/5 |
  | Opus 5.5 | `mvcc-lsm-compaction` | 0/5 → 4/5 |
  | Opus 5.5 | `cli-2ph-simplex` | 0/5 → 5/5 |
  | Opus 5.5 | `gsea-proteomics` | 0/5 → 4/5 |

  In the traced runs, the high-effort attempts reproduced the bug first, fuzzed, compared against brute-force references, and tried alternative data preparations.
- **Interpretation**: effort reduces *missed-edge-case* failures but "does not fix when the model has the wrong approach".

**Author's practice** (anecdotal, not measured):
- On underspecified builds, effort changes scope a lot (low gives a log plus a graph; max adds a heat chart). Given a detailed spec, levels converge.
- One design task took 1 minute at low effort and 28 minutes at max.
- Recommended loop: have Claude interview you for a spec, implement on **low**, review, then verify/test on **high**.
- Rule of thumb:
  - **Low**: brainstorming and easy changes.
  - **Medium**: most feature work.
  - **High**: brownfield bug fixes and verification-heavy work.
  - **Max**: fully autonomous hard problems.

## Concepts & entities

- [[reasoning-effort]], [[test-time-compute]], [[adaptive-thinking]]
- [[terminal-bench]] (benchmark), [[plan-execute-verify-loop]] (the interview → build → verify loop)
- [[claude-code]], [[thariq-shihipar]], [[anthropic]]

## What it does not show

- No independent replication. All figures are vendor-internal, and several worked examples come from single traced runs, some at intermediate effort settings.
- Category pass rates pool the "two lowest" vs. "three highest" settings, so they are not clean per-level comparisons.
- No cost/latency-adjusted comparison: tokens per level appear only in an unlabelled chart.

> [!warning] Contradiction
> [[claude-effort-and-ultracode-docs-2026|The effort docs]] (June 2026, Opus 4.7/4.8) recommend **"starting at `xhigh`"** for coding/agentic work, with `high` as the default. This post (Sept 2026, Opus 5.5/Fable 5.1) recommends **medium** for "most of my regular software engineering work" and **low** for implementation inside an interview/verify loop. The two sources cover different model generations, and one is official guidance while the other is personal practice.

## References

- https://claude.dev/blog/spending-your-effort/
- Terminal-Bench 3.0 tasks: https://github.com/harbor-framework/terminal-bench/releases/tag/v3.0.0
- _Original source: `sources/using-claude-code-spending-your-effort-shihipar-2026.md`_
