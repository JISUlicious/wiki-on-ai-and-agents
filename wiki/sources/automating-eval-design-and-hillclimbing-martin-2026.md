---
title: "Automating eval design and hillclimbing with Claude"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - automating-eval-design-and-hillclimbing-martin-2026.md
authors:
  - Lance Martin
year: 2026
venue: "claude.dev (Anthropic developer blog)"
status: complete
importance: high
tags:
  - 2026
---

# Automating eval design and hillclimbing with Claude

**Lance Martin** ([[anthropic|Anthropic]]). Published 2026-09-28 on claude.dev.
**URL**: https://claude.dev/blog/automating-eval-design-and-hillclimbing/

## Summary

The post sets out principles for designing evals and for hill-climbing against them "without fooling yourself". It then shows how two new sub-commands of the `claude-api` [[agent-skills|skill]] in [[claude-code|Claude Code]] apply them. `/claude-api build-eval` interviews the user and builds the cases, grader, runner and results page in the codebase. `/claude-api hillclimb` makes one patch per round against a random train/test split and reverts any patch that looks like overfitting. Most of the post is **advice**. Measured results appear only in the two worked examples at the end, which are Anthropic-internal.

## Key points

- **Four marks of a good eval (advice):** tasks mirror production; scores rise with stronger models and more [[reasoning-effort|effort]]; the best model at the highest effort is well below 100%, and not because tasks are impossible; and run-to-run variance is low. A task that fails on every replicate is a sign it is ambiguous or broken.
- **Avoid adversarial sampling.** Picking cases because today's model fails them records that model's "failure fingerprint". Include a hard case only when you can say *why* it is hard. Production traffic can skew easy.
- **Grader choice:** use the cheapest grader that fits. That means a programmatic check for constrained outputs, otherwise [[llm-as-a-judge]] with a rubric of checkable claims (not a 1–5 scale). For baselines, use pairwise blind comparison in random order, with a judge model different from the one being tested. The skill runs the grader twice on the same output, checks for timeouts and truncation, and warns when the baseline is at ~95% or more.
- **Where to hill-climb:** choose surfaces that are cheap to change and whose effect can be attributed, such as prompts, skills and skill-trigger descriptions. Open-ended harness edits are a poor target. Cost is a good objective even when quality is saturated.
- **Overfitting defences:** a held-out test split; never pasting failing transcripts into the prompt; and keeping answers out of the model's reach to prevent [[reward-hacking]]. The hillclimber keeps a patch only if train *and* test improve. If train rises while test stays flat, it reverts. Before round one it checks that eval noise is smaller than the smallest gain you would act on. After 2–3 stalled rounds it sorts the remaining failures by root cause.
- **Example 1, cost (Anthropic-reported):** an internal support benchmark of 44 tickets (30 for search, 14 held out). The baseline was Opus 4.8 at high effort: 74.4% accuracy, 4.6¢ per ticket. After a prompt audit, Opus 5.5 at low effort scored 87.8% at 1.9¢, and Sonnet 5 at low effort scored 88.9% at ~1¢. Routing rules then lifted Sonnet 5 to 98.9% on the search set. **Held-out: 90.5% vs 78.6% at ~1/5 the cost.** The large drop from 98.9% to 90.5% shows how much the search split was overfit.
- **Example 2, quality (Anthropic-reported):** the claude-api skill's own eval rose from 66.1% to 87.9%. Gains came from eight missing features (74%), C# and Java type-table fixes (77%), and a "stale API shapes" table steering Claude from fixed-budget thinking to [[adaptive-thinking]] (80%). Repairing flawed tasks and graders brought it to ~88%. The post does not say whether 87.9% is a held-out score.

## Concepts & entities

- [[agent-evaluation]] · [[skill-evaluation]] · [[llm-as-a-judge]] · [[verifier]] — eval design and grading.
- [[skill-optimization]] · [[skill-routing]] — hill-climbing a SKILL.md and its triggering is the same loop that literature automates.
- [[benchmark-contamination]] · [[reward-hacking]] — the leakage and overfitting failure modes.
- [[claude-code]] · [[agent-skills]] · [[anthropic]].
- Companion: [[demystifying-evals-for-ai-agents-anthropic-2026]], which the post cites ("read a sample of scored transcripts").

## What it does not show

- Both examples are small (44 tickets, one skill eval) and internal, and no confidence intervals are printed in the post.
- There is no comparison against a human-designed eval or a non-Claude hillclimber.
- Part of the cost saving comes from Opus 5.5 list prices (20% lower for tokens, 60% lower for cache reads), not from the method.

## References

- https://claude.dev/blog/automating-eval-design-and-hillclimbing/
- claude-api skill: https://github.com/anthropics/skills/tree/main/skills/claude-api
- _Original source: `sources/automating-eval-design-and-hillclimbing-martin-2026.md`_
