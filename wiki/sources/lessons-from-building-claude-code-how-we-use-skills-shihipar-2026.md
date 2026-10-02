---
title: "Lessons from building Claude Code: How we use skills (Shihipar 2026)"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - lessons-from-building-claude-code-how-we-use-skills-shihipar-2026.md
authors:
  - [[thariq-shihipar]]
year: 2026
venue: "claude.dev (Anthropic developer blog)"
publisher: Anthropic
status: complete
importance: high
tags:
  - 2026
---

# Lessons from building Claude Code: How we use skills

**Author**: [[thariq-shihipar|Thariq Shihipar]] · **Date**: 2026-06-03 · **Publisher**: [[anthropic|Anthropic]] (claude.dev, "Engineering")
**URL**: https://claude.dev/blog/lessons-from-building-claude-code-how-we-use-skills/

## Summary

An internal-practice report on [[agent-skills|Agent Skills]] in [[claude-code|Claude Code]], based on "hundreds of them in active use" at Anthropic. The post makes four contributions:
1. A **nine-category taxonomy** of internal skills: library/API reference, product verification, data fetching/analysis, business automation, scaffolding/templates, code quality/review, CI/CD/deployment, runbooks, and infrastructure ops. The best skills "fit cleanly into one" category.
2. Authoring tips.
3. Distribution advice.
4. A measurement practice.

It insists a skill is "a folder, not just a markdown file": scripts, assets, data, config and logs are first-class.

## Key points

- **Verification skills** "have had the most measurable impact" on output quality internally. The author says it can be worth an engineer spending a week on them. This is an Anthropic claim; no numbers are given.
- **Authoring tips**:
  - Don't state the obvious.
  - The **Gotchas section** is "the highest-signal content", grown over time from observed failures.
  - Avoid railroading: state the goal and constraints, not every step.
  - Write the `description` *for the model*, as trigger conditions rather than a summary. This is the [[skill-routing]] surface.
- **File system as progressive disclosure**: SKILL.md acts as a hub with a symptom→file table pointing to spokes (`references/api.md`, `assets/` templates).
- **Stateful skills**:
  - A `config.json` setup flow, which asks the user when unconfigured (optionally via AskUserQuestion).
  - Memory via append-only logs, JSON or SQLite under `${CLAUDE_PLUGIN_DATA}`.
  - Bundled helper libraries that Claude composes into one-off scripts.
- **On-demand hooks**: skills can register hooks active only for the session after invocation. Examples: `/careful` blocks `rm -rf`, `DROP TABLE` and force-push via a PreToolUse Bash matcher; `/freeze` blocks edits outside one directory.
- **Distribution**: commit to `./.claude/skills` for small teams. At scale, use a plugin marketplace, because "every skill that is checked in also adds a little bit to the context". Curation is organic: sandbox folder → traction → PR into the marketplace. Skills compose by name reference, with no native dependency management.
- **Measurement**: a PreToolUse hook logs skill usage company-wide to find popular and *undertriggering* skills.

## Concepts & entities

- [[agent-skills]], [[skill-md-format]], [[skill-routing]], [[skill-selection]], [[skill-library]], [[skill-evaluation]]
- [[hooks-in-modern-agents]] (skill-scoped, on-demand hooks), [[guardrailing]] (`/careful`)
- [[claude-code]], [[thariq-shihipar]], [[anthropic]]; earlier Anthropic framing in [[anthropic-engineering-skills-2025]]

## Relation to the progressive-disclosure evidence

The post presents file-system disclosure as best practice. It does not claim this improves accuracy. The [[agent-skills]] page now says:

> "Disclosure helps when the agent would otherwise navigate badly, and adds nothing measurable under a strong harness … adding levels of *always-loaded* descriptions can hurt badly"

That is per [[is-progressive-disclosure-all-you-need-he-2026|He et al. 2026]]. The post's advice largely **agrees**:
- Its hub-and-spokes design is one layered skill, which matches He et al.'s "one level" prescription.
- It warns that each checked-in skill "adds a little bit to the context", consistent with the always-loaded-cost finding.

The tension is only in emphasis. The post calls the whole file system "a form of context engineering and progressive disclosure" without stating when disclosure fails to help.

## What it does not show

- No quantitative results. The taxonomy, the "most measurable impact" claim and the tips are internal experience, not controlled comparisons.
- No data on triggering accuracy or on how many skills a session can hold before routing degrades.

## References

- https://claude.dev/blog/lessons-from-building-claude-code-how-we-use-skills/
- Usage-logging hook gist: https://gist.github.com/ThariqS/24defad423d701746e23dc19aace4de5
- _Original source: `sources/lessons-from-building-claude-code-how-we-use-skills-shihipar-2026.md`_
