---
title: SKILL.md Format (Agent Skills)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - claude-code.md
  - opencode.md
  - pi-mono.md
  - hermes-agent.md
  - openclaw.md
status: complete
importance: high
---

# SKILL.md Format (Agent Skills)

A simple markdown-with-frontmatter file format for **packaging reusable agent capabilities** ("skills") that any LLM-agent runtime can load. Originated as **Anthropic's Agent Skills v1** specification (deployed in [[claude-code|Claude Code]]) and has become the **de-facto cross-agent interop standard** as of 2026.

## Format

A skill is a directory containing at minimum a `SKILL.md` file:

```markdown
---
name: my-skill
description: One-line description of what this skill does
---

# Skill body — instructions the LLM follows when this skill is invoked.

When the user asks for X, do these steps:
1. ...
2. ...
```

The frontmatter typically declares a `name`, `description`, and may include other metadata (target tools, model preferences, etc.). The markdown body is the instruction set the LLM reads when the skill is activated.

Skills can also contain auxiliary files (templates, scripts, sub-instructions) that the markdown references — but the SKILL.md is the entry point.

## Cross-agent adoption (2026 status)

All five documented 2026 agents read SKILL.md:

- **[[claude-code|Claude Code]]**: walks `~/.claude/skills/` and project-local `.claude/skills/`. Origin of the spec.
- **[[opencode]]**: walks `~/.claude/skills/`, `.agents/skills/`, project dirs. Explicit Claude-Code compatibility.
- **[[pi-mono|pi]]**: walks `~/.pi/` and `~/.agents/`. Refers to "Agent Skills v1" interop.
- **[[hermes-agent]]**: Anthropic-inspired plus agentskills.io compatibility. Bundled + optional skills.
- **[[openclaw]]**: inherits from pi.

The convergence is not formal standardization but **organic adoption** — Claude Code published the spec and the format was simple enough that other agents incorporated it without modification.

## Why this format won

- **Markdown + frontmatter is already universal** — every editor, every LLM, every developer reads it.
- **No new runtime required** — a skill is just a file the LLM reads. No registry, no protocol, no toolchain.
- **Composable**: a directory of skill subdirectories. Walk the tree, load what's relevant.
- **Versionable**: skills go in git like everything else.
- **Inspectable**: humans can read the body and understand what the skill does.

The trade-off: skills are *prompt-engineering artifacts*, not executable code. They tell the LLM what to do; the LLM still needs to do it. This works well for instruction-following on capable models and is brittle on smaller / less-aligned ones.

## Relation to MCP

[[tool-use|MCP]] (Model Context Protocol) is the parallel standard for **tool integration** — connecting an agent to external services. SKILL.md is for **instruction packaging** — telling the agent how to use what it already has. The two are complementary, not competing.

## References

- [[claude-code-agent-doc-2026]]
- [[opencode-agent-doc-2026]]
- [[pi-mono-agent-doc-2026]]
- [[hermes-agent-doc-2026]]
- [[openclaw-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]
