---
title: Subagents
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - claude-code-changelog-2026-10-06.md
  - effective-context-engineering-anthropic-2025.md
  - building-effective-agents-anthropic-2024.md
status: draft
importance: medium
---

# Subagents

A lead agent delegates a focused task to a child agent that runs in its **own clean context window**. The child returns a condensed result, "often 1,000–2,000 tokens" per [[effective-context-engineering-anthropic-2025|Anthropic]].

Subagents are both a context-management technique and the run-time form of the [[agentic-workflow-patterns|orchestrator-workers]] pattern.

## Why

- **Context isolation.** Exploration that would fill the lead agent's context stays in the child, and only the distilled summary comes back. See [[context-engineering]] and [[context-rot]].
- **Specialization.** Each child gets its own instructions, tools and model.
- **Parallelism.** Independent subtasks run at the same time. Anthropic's multi-agent research system reported "a substantial improvement over single-agent systems on complex research tasks."

## Claude Code timeline

Dates are npm publish dates, from [[claude-code-changelog-2026]]:

| Date | Version | Change |
|---|---|---|
| 2025-04-18 | 0.2.74 | The `Task` tool can now write files and run bash, so delegated work can act |
| **2025-07-24** | 1.0.60 | **Custom subagents**: "Run /agents to get started" |
| 2025-07-28 | 1.0.62 | @-mention invocation of custom agents |
| 2025-11-17 | 2.0.43 | Subagents can auto-load skills (frontmatter `skills` field) |
| 2026-05 | — | [[dynamic-workflows]]: Claude writes scripts that spawn tens to hundreds of subagents |

## Caveats

- Multi-agent setups cost more tokens and can lose information at the hand-offs.
- Whether they beat a single agent *at matched compute* is contested; see [[multi-agent]].

## Related

[[multi-agent]] · [[agentic-workflow-patterns]] · [[dynamic-workflows]] · [[claude-code]] · [[context-engineering]] · [[babyagi]], whose 2023 executor/creator/prioritizer roles were an early role split.

## References

- [[claude-code-changelog-2026]]
- [[effective-context-engineering-anthropic-2025]]
- [[building-effective-agents-anthropic-2024]]
