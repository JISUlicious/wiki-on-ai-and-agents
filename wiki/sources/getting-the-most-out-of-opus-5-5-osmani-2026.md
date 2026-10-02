---
title: "Getting the most out of Opus 5.5 in Claude and Claude Code"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - getting-the-most-out-of-opus-5-5-osmani-2026.md
authors:
  - Addy Osmani
year: 2026
venue: "claude.dev (Anthropic developer blog)"
publisher: Anthropic
status: complete
importance: medium
tags:
  - 2026
---

# Getting the most out of Opus 5.5 in Claude and Claude Code

**Source**: `sources/getting-the-most-out-of-opus-5-5-osmani-2026.md` (official [[anthropic|Anthropic]] developer blog)
**Author**: Addy Osmani (Anthropic)
**Date**: 2026-09-22
**URL**: https://claude.dev/blog/getting-the-most-out-of-opus-5-5/

## Summary

A user-facing prompting guide for **Claude Opus 5.5** in the Claude apps and [[claude-code|Claude Code]]. Its framing is that Opus 5.5 "works for longer on its own, it tells you plainly what it did, and it thinks before every reply". Almost all of the post is **advice**. Its few capability statements are **Anthropic-reported comparisons with Opus 5**, backed by internal testing or unnamed "early testers", with no figures.

## Key points

- **Hand over the whole task (advice)**: state what "done" looks like and when to stop and ask, then let it run. The post claims its "biggest gains are on multi-step work" and that early testers ran coding tasks "for hours with little oversight". These are Anthropic-reported and unquantified.
- **Drop "think carefully" (advice plus an internal result)**: Opus 5.5 "always thinks before it replies, and it decides how much". In Anthropic's testing in a chat product, removing such a line made replies start sooner "with no clear drop in quality". To change thinking depth, change effort instead.
- **Steer stops through `CLAUDE.md` (advice)**: Opus 5.5 sometimes stops on a long run to report or offer to continue. The post gives a `CLAUDE.md` rule for when to keep going and when to stop before destructive actions, and says to keep permission prompts on.
- **Fan out to subagents and keep task state in a file (advice)**: split audits and migrations across [[multi-agent|subagents]] and verify each one's evidence. Keep a `TASKS.md` checklist because context compaction loses scrollback.
- **Design prompts (advice)**: without direction the model "falls back on a few default styles". The post says naming specific patterns to avoid works better than "avoid a generic look".
- **Quality claims (Anthropic-reported, not measured here)**:
  - It reads charts and screenshots "more accurately than Opus 5".
  - It has better attention to detail in long documents.
  - Its files "need less editing".
  - One early tester said its lowest effort "caught more bugs than Opus 5 at high effort, with fewer false alarms".
- **Safeguards**: "the first Opus model to launch with Fable-level bio and cyber safeguards". Most flagged messages move to an older model. Requests to reproduce internal reasoning can be declined as a flag category.
- **Fast mode**: available at launch "as a research preview" and "costs more per token" (see [[what-a-task-costs-on-opus-5-5-osmani-2026]] for the price).

## Concepts & entities

- [[claude]] · [[anthropic]] · [[claude-code]] (`CLAUDE.md` rules, `/model`, `/fast`, `/config`).
- [[reasoning-effort]] · [[adaptive-thinking]]: always-on thinking replaces "think step by step" prompting (cf. [[chain-of-thought]]).
- [[multi-agent]] · [[dynamic-workflows]]: subagent fan-out with evidence checking.
- [[context-assembly]]: `CLAUDE.md` as standing instructions, and the file-based task list as state that survives compaction.
- [[same-task-different-work-weinberger-2026]] is independent evidence on the "think carefully" point. On Sonnet 5, `deep_thinking` incantations had the mildest effect, which fits the adaptive-thinking explanation.

## What it does not show

- No numbers at all: no benchmarks, task counts or tester counts. "Early testers" and "our testing" are anecdotes.
- The flag and fallback behaviour is described, but not how often legitimate work gets flagged.

## References

- https://claude.dev/blog/getting-the-most-out-of-opus-5-5/
- Companions: [[what-a-task-costs-on-opus-5-5-osmani-2026]] · [[building-with-claude-sonnet-5-5-osmani-2026]]
- _Original source: `sources/getting-the-most-out-of-opus-5-5-osmani-2026.md`_
