---
title: Context Assembly
type: concept
created: 2026-05-29
updated: 2026-10-04
sources:
  - code-as-agent-harness-ning-2026.md
status: draft
importance: medium
tags: []
---

# Context Assembly

**Context assembly** is the L2 layer of the [[agent-three-layer-model|three-layer agent architecture]]: the machinery that constructs each turn's prompt before it reaches the model. Rather than a static system prompt, a turn's context is *assembled* on demand from many sources — auto-loaded project files, retrieved snippets, activated [[agent-skills|skills]], dynamic templates, and the base system prompt — and the quality of this assembly largely determines agent behavior. Although referenced by many pages as the locus of "what the model actually sees," it lacked a dedicated page until now.

- **Auto-loaded files** — e.g. `CLAUDE.md`-style instructions injected every turn.
- **Retrieved snippets** — relevant context pulled via search or embeddings ([[in-context-learning]] in practice).
- **Activated skills** — progressively disclosed skill bodies loaded only when a descriptor matches the task.
- **Dynamic templates + tool surfaces** — system prompt plus tool/[[mcp]] schemas stitched into the window.

In the three-layer model, context assembly is one of the six L2 design problems, alongside control flow, action surface, [[memory-management|memory management]], persistence and the harness. It sits between the model's own capabilities (L1, above all [[in-context-learning]], which is what lets an assembled prompt change behaviour at all) and the concrete mechanisms that implement it (L3: `CLAUDE.md` loading, skill activation, retrieval, MCP schemas). Its job is to turn stored memory and available capabilities into a single coherent prompt.

## Related

- [[agent-three-layer-model]] — the architecture this is the L2 of.
- [[agent-skills]] — a major source of assembled context.
- [[memory-management]] — supplies the persistent material to assemble.

## References

- [[code-as-agent-harness-ning-2026]]

## Related 2026 sources

- [[new-rules-of-context-engineering-claude-5-shihipar-2026]] — Shihipar: Anthropic cut 80%+ of Claude Code's system prompt for Claude 5 models with no measured loss
- [[lessons-from-building-claude-code-prompt-caching-is-everything-shihipar-2026]] — How Claude Code is built around prefix caching: static-first layout, no mid-session swaps, cache-safe compaction
