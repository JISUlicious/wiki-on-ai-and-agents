---
title: claude.dev
type: entity
created: 2026-10-02
updated: 2026-10-02
sources:
  - getting-started-with-claude-code-mods-osmani-2026.md
founders:
  - [[anthropic]]
status: draft
importance: medium
tags:
  - product
---

# claude.dev

[[anthropic|Anthropic]]'s developer blog, "sharing tips, tricks, and POVs from Anthropic's developers", at [claude.dev](https://claude.dev). Its posts are filed under Agents, Engineering, Playbooks, Skills and Tutorials. It also hosts a gallery of [[claude-code-mods|Claude Code mods]] and a `/terminal/` view that presents the blog as a shell. It publishes an RSS feed.

Its main authors in this wiki are [[thariq-shihipar]] (on how Claude Code itself is built) and Addy Osmani (model launch guides and tutorials).

> [!warning] A vendor writing about its own products
> These posts are useful because they come from the people who build the systems. But every performance, cost and benchmark figure in them is Anthropic-reported and mostly has no published methodology. The wiki's source pages label figures as such and keep advice separate from measurement.

## Posts in this wiki

**Claude 5.5 models and cost**
- [[building-with-claude-sonnet-5-5-osmani-2026]] — Sonnet 5.5 launch guide and migration changes (see [[claude-sonnet-5-5]])
- [[getting-the-most-out-of-opus-5-5-osmani-2026]] — prompting Opus 5.5 (see [[claude-opus-5-5]])
- [[what-a-task-costs-on-opus-5-5-osmani-2026]] — a worked cost model for an agentic task, and what "40% cheaper" means
- [[using-claude-code-spending-your-effort-shihipar-2026]] — what higher effort buys on Terminal-Bench 3.0 (verification, not better approaches)

**How Claude Code is built**
- [[seeing-like-an-agent-shihipar-2026]] — tool-design lessons (April 2026)
- [[lessons-from-building-claude-code-prompt-caching-is-everything-shihipar-2026]] — the harness designed around prefix caching
- [[lessons-from-building-claude-code-how-we-use-skills-shihipar-2026]] — Anthropic's internal skill practice
- [[new-rules-of-context-engineering-claude-5-shihipar-2026]] — 80%+ of the system prompt cut for Claude 5 models
- [[claude-code-dynamic-workflows-harness-2026]] — dynamic workflows (ingested earlier from its claude.com copy)
- [[getting-started-with-claude-code-mods-osmani-2026]] — mods

**Engineering and practice**
- [[how-we-made-claude-ai-3x-faster-wang-2026]] — a Claude-driven web-performance sprint
- [[automating-eval-design-and-hillclimbing-martin-2026]] — eval design and overfit-safe hill-climbing
- [[unreasonable-effectiveness-of-html-shihipar-2026]] — HTML over Markdown for human-facing agent output (opinion)

## Related

- [[anthropic]] · [[claude-code]] · [[claude]] · [[thariq-shihipar]]
