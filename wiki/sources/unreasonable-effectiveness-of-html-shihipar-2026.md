---
title: "Using Claude Code: The unreasonable effectiveness of HTML"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - unreasonable-effectiveness-of-html-shihipar-2026.md
authors:
  - [[thariq-shihipar]]
year: 2026
venue: "claude.dev (Anthropic developer blog)"
status: complete
importance: medium
tags:
  - 2026
---

# Using Claude Code: The unreasonable effectiveness of HTML

**[[thariq-shihipar|Thariq Shihipar]]** ([[anthropic|Anthropic]]). Published 2026-05-20 on claude.dev. The post says it states his "personal opinions – and affinity".
**URL**: https://claude.dev/blog/using-claude-code-the-unreasonable-effectiveness-of-html/

## Summary

An opinion piece arguing that agents like [[claude-code|Claude Code]] should produce **HTML files instead of Markdown** when writing for humans: specs, plans, reports, code reviews and prototypes. Markdown's main advantage, easy hand-editing, matters less when the human asks Claude to make the edits. Meanwhile, Markdown's limits (low information density, hard to read past ~100 lines, awkward to share) get worse as agents produce longer outputs. The post is **advice and personal experience with no measurements**. It comes with a template gallery (thariqs.github.io/html-effectiveness, github.com/anthropics/html-effectiveness).

## Key points

- **Why HTML:** information density (tables, CSS, SVG, interactive JS, canvases); visual structure (tabs, navigation, mobile layouts); easy sharing as a link; **two-way interaction**; and the context Claude Code can draw on (file system, [[mcp|MCP]] servers, browser, git history). Without HTML, models fall back on ASCII diagrams and unicode "colour swatches".
- **Use cases with example prompts:** (1) specs, planning and exploration, as a "web of HTML files" handed to a fresh implementation session and to the verifier; (2) code review, with rendered diffs and severity-tagged margin notes; (3) design and prototypes with sliders (Claude Design is built on HTML); (4) reports and explainers; (5) **throwaway custom editors**.
- **The export trick:** end every throwaway editor with a "copy as JSON / prompt / diff" button, so that what the user does in the UI goes back to the agent as text. This is the main mechanism for keeping the human in the loop.
- **Getting started:** just ask Claude to "make an HTML file". Build a skill only once a pattern repeats.
- **Cost (FAQ):** HTML uses more tokens than Markdown. The author says the extra expressiveness, and the higher chance he will actually read the output, are worth it, and that with Opus 4.7's 1M-token window the overhead "is not really noticeable". This is a judgement, not a measurement.
- **Thesis:** the real benefit is staying engaged with the agent's choices as it takes on more work, instead of skimming long plans.

## Concepts & entities

- [[claude-code]] · [[anthropic]] · [[thariq-shihipar]].
- [[structured-outputs]] — the opposite direction: machine-parseable output for the harness, versus rich output for humans.
- [[agent-skills]] · [[skill-md-format]] — the wiki's skill pages assume Markdown (SKILL.md) as the agent-facing format. This post is about *human-facing* output, so there is no conflict.
- [[long-context-llm]] — the stated reason the token overhead is acceptable.

## What it does not show

- No reading-comprehension or review-quality comparison between HTML and Markdown. "More likely to be read" is the author's experience.
- The token cost is not quantified. HTML is harder to diff and hand-edit, and opening generated JS raises safety questions. The post does not discuss either.
- It does not address agent-to-agent handoff, where Markdown is compact and still dominant.

## References

- https://claude.dev/blog/using-claude-code-the-unreasonable-effectiveness-of-html/
- _Original source: `sources/unreasonable-effectiveness-of-html-shihipar-2026.md`_
