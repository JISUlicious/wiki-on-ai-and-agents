---
title: Context Assembly
type: concept
created: 2026-05-29
updated: 2026-05-29
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

Context assembly sits above the harness (L1) and below the agent's reasoning loop (L3), turning [[memory-management|stored memory]] and available capabilities into a single coherent prompt.

## Related

- [[agent-three-layer-model]] — the architecture this is the L2 of.
- [[agent-skills]] — a major source of assembled context.
- [[memory-management]] — supplies the persistent material to assemble.

## References

- [[code-as-agent-harness-ning-2026]]
