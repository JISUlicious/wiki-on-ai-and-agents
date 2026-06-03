---
title: "A Harness for Every Task: Dynamic Workflows in Claude Code"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - claude-code-dynamic-workflows-harness-2026.html
authors:
  - [[thariq-shihipar]]
  - [[sid-bidasaria]]
introduces:
  - [[dynamic-workflows]]
year: 2026
venue: claude.com blog
publisher: Anthropic
tags:
  - 2026
status: complete
importance: high
---

# A Harness for Every Task: Dynamic Workflows in Claude Code

**Source**: `sources/claude-code-dynamic-workflows-harness-2026.html` (official [[anthropic|Anthropic]] blog)
**Authors**: [[thariq-shihipar|Thariq Shihipar]] and [[sid-bidasaria|Sid Bidasaria]] (members of technical staff, Anthropic)
**Date**: 2026-06-02
**URL**: https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code

## Summary

The engineering deep-dive on [[dynamic-workflows]]. Its thesis: [[claude-code|Claude Code]] can now **write and orchestrate its own multi-agent harness on the fly**, moving beyond the default coding-focused harness to handle research, security analysis, agent teams, and code review more natively. A workflow is a JavaScript file with special functions to spawn and coordinate [[multi-agent|subagents]], choosing each one's model, whether it runs in an isolated git **worktree**, and resuming from a journal if interrupted.

The post's most cited contribution is its diagnosis of *why* a single context window fails on hard tasks — **agentic laziness, self-preferential bias, and goal drift** — and its catalog of orchestration **patterns** and **use cases** that combat those failure modes by giving each subagent a clean context and an independent verifier. This is the conceptual companion to the [[claude-code-dynamic-workflows-2026|May 28 announcement]].

## Key points

- **Why workflows** — extended work in one context window invites three failure modes: **agentic laziness** (declaring a multi-part job done after partial progress, e.g. 35/50 review items), **self-preferential bias** (preferring one's own results when verifying against a rubric), and **goal drift** (lossy summarization dropping edge-case / "don't do X" constraints). Separate subagents with isolated goals and *independent* verification fix this.
- **Dynamic vs. static** — static workflows (Claude Agent SDK / `claude -p`) are generic; with [[claude-opus-4-8|Claude Opus 4.8]], Claude writes a **custom harness tailored to the task**.
- **Patterns** — classify-and-act, fan-out-and-synthesize (barrier merge of structured outputs), adversarial verification, generate-and-filter, tournament (pairwise judging), and loop-until-done (stop on no-new-findings rather than fixed passes).
- **Use cases** — migrations/refactors (the Bun Zig→Rust rewrite, worktree-per-fix + adversarial review-and-merge), deep research (the `/deep-research` skill), deep verification, sorting via pairwise-comparison pipelines, memory/rule-adherence (mine sessions → distill `CLAUDE.md` rules with per-rule verifier + skeptic persona), root-cause investigation (disjoint-evidence hypothesis agents + verifier/refuter panels), triage at scale (with a **quarantine** pattern for untrusted content), evals, exploration/taste, and model/intelligence routing.
- **When not to use** — workflows cost significantly more tokens and aren't needed for most ordinary coding; "most traditional coding tasks don't require a panel of five reviewers."
- **Tips** — detailed pattern-based prompting; combine with `/loop` (intervals) and `/goal` (hard completion); set token budgets ("use 10k tokens"); save with "s" to `~/.claude/workflows` or distribute via an [[agent-skills|Agent Skill]] (`.js` in the skill folder, referenced from `SKILL.md`, optionally as a template).

## Concepts & entities

- [[dynamic-workflows]] — the concept this post defines in depth.
- [[claude-code]] — host runtime; [[multi-agent]] — subagent substrate; [[code-as-harness]] — self-authored-harness framing.
- [[guardrailing]] / [[indirect-prompt-injection]] — the "quarantine" pattern's security rationale.
- [[agentic-misalignment]] — self-preferential bias in self-verification.
- [[thariq-shihipar]], [[sid-bidasaria]] — authors; [[anthropic]] — publisher.

## Notable quotes

> "Each summarization step is lossy, and details like edge-case requirements or 'don't do X' constraints can get lost."

## References

- [[dynamic-workflows]]
- [[claude-code-dynamic-workflows-2026]] — the announcement companion.
- _Original source: `sources/claude-code-dynamic-workflows-harness-2026.html`_
