---
title: SKILL.md Format (Agent Skills)
type: concept
created: 2026-05-14
updated: 2026-05-31
sources:
  - claude-code.md
  - opencode.md
  - pi-mono.md
  - hermes-agent.md
  - openclaw.md
  - anthropic-agent-skills-2025.md
  - agentskills-io-spec-2025.md
  - openai-codex-skills-2025.md
status: complete
importance: high
---

# SKILL.md Format (Agent Skills)

> [!note] Canonical concept
> This page covers the **file format** as a cross-agent interop artifact. For Anthropic's full architecture (three-level progressive disclosure, cross-platform availability, governance, open-standard at agentskills.io), see **[[agent-skills]]**. For the normative spec, see **[[agentskills-io-spec-2025]]**.

A simple markdown-with-frontmatter file format for **packaging reusable agent capabilities** ("skills") that any LLM-agent runtime can load. Originated as **Anthropic's Agent Skills** specification (announced 2025-10-16 — see [[anthropic-agent-skills-2025]] — and deployed in [[claude-code|Claude Code]]), formalized as the open standard at **[[agentskills-io-spec-2025|agentskills.io]]** (2025-12-18), and now the **de-facto cross-agent interop standard** with 40+ adopters as of 2026.

## Format

A skill is a directory containing at minimum a `SKILL.md` file:

```markdown
---
name: my-skill
description: One-line description of what this skill does and when to use it
---

# Skill body — instructions the LLM follows when this skill is invoked.

When the user asks for X, do these steps:
1. ...
2. ...
```

### Frontmatter schema (per the [[agentskills-io-spec-2025|agentskills.io spec]])

> [!warning] Correction
> Earlier versions of this wiki described the frontmatter as just `name` + `description`. The formal open standard defines **six fields**, with strict naming rules:

| Field | Required | Constraints |
|---|---|---|
| `name` | yes | 1–64 chars; **must match the parent directory name**; no leading/trailing hyphen; no consecutive hyphens (`--`) |
| `description` | yes | ≤1024 chars; states what the skill does *and when to use it* (this is what L1 disclosure surfaces) |
| `license` | no | SPDX-style license identifier |
| `compatibility` | no | ≤500 chars; runtime/version compatibility notes |
| `metadata` | no | arbitrary key-value map |
| `allowed-tools` | no | space-separated tool list (marked **Experimental**) |

Runtimes extend this freely: [[claude-code|Claude Code]] adds ~15 further fields (`disable-model-invocation`, `user-invocable`, `context: fork`, `agent`, `model`, `paths`, `hooks`, `argument-hint`, `when_to_use`, `disallowed-tools`, etc.). The agentskills.io spec ships a reference validator, `skills-ref validate ./my-skill`. Token budgets: metadata ≈100 tokens (L1); SKILL.md body recommended <5000 tokens (L2).

The markdown body is the instruction set the LLM reads when the skill is activated. Skills can also contain auxiliary files (templates, scripts, sub-instructions) that the markdown references — but the SKILL.md is the entry point.

## Cross-agent adoption (2026 status)

By 2026 the format spans both the wiki's documented agents and the major vendors. The agentskills.io spec lists **40+ adopters**.

**The five documented 2026 agents** all read SKILL.md:

- **[[claude-code|Claude Code]]**: walks `~/.claude/skills/` and project-local `.claude/skills/`. Origin of the spec; custom commands have merged into skills.
- **[[opencode]]**: walks `~/.claude/skills/`, `.agents/skills/`, project dirs. Explicit Claude-Code compatibility.
- **[[pi-mono|pi]]**: walks `~/.pi/` and `~/.agents/`. Refers to "Agent Skills v1" interop.
- **[[hermes-agent]]**: Anthropic-inspired plus agentskills.io compatibility. Bundled + optional skills.
- **[[openclaw]]**: inherits from pi.

**Cross-vendor convergence** (the significant 2026 development):

- **[[openai-codex-skills-2025|OpenAI Codex]]** independently shipped `SKILL.md` — same directory + name/description + progressive disclosure model. Uses `.agents/skills` (vs `.claude/skills`) and an `agents/openai.yaml` metadata file. The strongest evidence the format is a genuine interop standard, not Anthropic-only.
- **Google Gemini CLI Extensions** bundle what Google's docs call "Agent Skills — instructions and workflows the model activates only when needed."
- Other listed adopters: Cursor, GitHub Copilot, VS Code, Goose, [[letta|Letta]].

The convergence began as **organic adoption** — Claude Code published the format and it was simple enough to copy — and was then formalized into the [[agentskills-io-spec-2025|agentskills.io open standard]] (Dec 2025). See [[agent-skills-cross-vendor]] for the full vendor comparison (including the manifest/config equivalents from Microsoft, AWS, and the hosted-config GPTs/Gems).

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
