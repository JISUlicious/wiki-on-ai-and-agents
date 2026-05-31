---
title: "Agent Skills Specification (agentskills.io)"
type: source
created: 2026-05-29
updated: 2026-05-29
sources:
  - agentskills-io-spec-2025.md
authors:
  - "[[anthropic]]"
year: 2025
introduces:
  - "[[skill-md-format]]"
tags:
  - 2025
  - spec
status: complete
importance: high
---

# Agent Skills Specification (agentskills.io)

**URL:** https://agentskills.io/specification — published 2025-12-18, maintained by [[anthropic]] as an open standard (GitHub: `agentskills/agentskills`).

## Summary

This is the **formal, normative specification** for the [[skill-md-format|SKILL.md format]] (a.k.a. [[agent-skills|Agent Skills]]). Where the wiki previously treated the format as a loosely-converged convention that "originated as Anthropic's Agent Skills v1," agentskills.io establishes it as a **published open standard** with a precise schema, a reference validator, and an explicit governance home open to ecosystem contributions. It is THE reference to cite for what is and is not a conformant skill.

The single most important correction this page makes: a skill's frontmatter has **six fields, not two.** The wiki's prior understanding ("a `name`, `description`, and may include other metadata") under-specifies the standard. The spec defines two **required** fields (`name`, `description`) and four **optional** ones (`license`, `compatibility`, `metadata`, `allowed-tools`), each with concrete constraints. It also imposes strict naming rules on `name` — 1–64 chars, lowercase alphanumeric plus hyphens, no leading/trailing hyphen, no consecutive hyphens (`--`), and crucially **the `name` must match the parent directory name**.

The spec is also significant as an **interoperability standard rather than a single-vendor format.** It lists 40+ adopters across the ecosystem — including Cursor, GitHub Copilot, VS Code, OpenAI Codex, Gemini CLI, Goose, and Letta — making SKILL.md a genuine cross-vendor format on par with [[mcp|MCP]] (instruction packaging vs. tool integration; the two are complementary). It quantifies the progressive-disclosure model that motivates the format: skill metadata (`name` + `description`) costs **~100 tokens** and is loaded at startup for all skills, while the full SKILL.md body is loaded only on activation and is **recommended to stay under 5000 tokens** (and under ~500 lines), with `scripts/`, `references/`, and `assets/` loaded only as needed.

## Frontmatter schema

A skill is a directory containing at minimum a `SKILL.md` file (YAML frontmatter + Markdown body). The frontmatter fields:

| Field | Required? | Constraints |
|---|---|---|
| `name` | **Yes** | 1–64 characters; lowercase alphanumeric (`a-z`, `0-9`) and hyphens only; **must not** start or end with a hyphen; **must not** contain consecutive hyphens (`--`); **must match the parent directory name**. |
| `description` | **Yes** | 1–1024 characters; non-empty. Should describe both *what* the skill does and *when* to use it, with keywords that help agents identify relevant tasks. |
| `license` | No | License name or a reference to a bundled license file (e.g. `Apache-2.0`, or `Proprietary. LICENSE.txt has complete terms`). |
| `compatibility` | No | 1–500 characters if present. Declares environment requirements (intended product, required system packages, network access, etc.). Most skills omit it. |
| `metadata` | No | Arbitrary key→value mapping (string→string) for client-defined properties not covered by the spec. Recommended to namespace keys to avoid conflicts. |
| `allowed-tools` | No | Space-separated string of pre-approved tools the skill may use (e.g. `Bash(git:*) Bash(jq:*) Read`). **Experimental** — support varies between agent implementations. |

Minimal valid skill:

```markdown
---
name: pdf-processing
description: Extract PDF text, fill forms, merge files. Use when handling PDFs.
---

# Skill body — instructions the agent follows once activated.
```

## Key points

- **Six frontmatter fields**, not two: required `name` + `description`; optional `license`, `compatibility`, `metadata`, `allowed-tools`.
- **`name` must match the parent directory name** — plus 1–64 chars, lowercase alphanumeric + hyphens, no leading/trailing hyphen, no `--`. This is the rule most likely to break naively authored skills.
- `description` allows up to **1024 characters** (the wiki had treated it as a one-line field).
- **Progressive disclosure is quantified**: metadata ≈ **~100 tokens** (loaded for all skills at startup); SKILL.md body **< 5000 tokens recommended** (loaded on activation); `scripts/` / `references/` / `assets/` loaded on demand.
- Ships a **reference validator**: `skills-ref validate ./my-skill` checks frontmatter validity and naming conventions.
- **40+ cross-vendor adopters** (Cursor, GitHub Copilot, VS Code, OpenAI Codex, Gemini CLI, Goose, Letta, and others) make this a real interop standard, not a single-vendor convention.
- File references should use relative paths and stay **one level deep** from SKILL.md; avoid deeply nested reference chains.

> [!note] This corrects the wiki's prior understanding
> Earlier wiki pages (notably [[skill-md-format]]) described skill frontmatter as essentially "`name`, `description`, and maybe some other metadata," and characterized the format as an organically-converged convention. The formal spec at agentskills.io supersedes that: the frontmatter is a **defined six-field schema** with strict constraints, `name` **must equal the parent directory name**, `description` goes up to 1024 chars, and the format is a **published open standard** with a reference validator (`skills-ref`) and 40+ cross-vendor adopters — not merely a de-facto convention. Treat this page as the normative reference; backfill [[skill-md-format]] and [[agent-skills]] to match.

## Entities

- [[anthropic]] — maintainer; released Agent Skills as an open standard.

## Concepts

- [[skill-md-format]] — the file format this spec formally defines.
- [[agent-skills]] — Anthropic's broader Agent Skills architecture (progressive disclosure, governance).
- [[mcp]] — parallel open standard for tool integration; complementary to SKILL.md's instruction packaging.

### Cross-vendor adopters (40+)

- [[claude-code]] — origin / first implementation.
- [[openai]] — OpenAI Codex.
- [[google-research]] — Gemini CLI.
- Others named in the spec include Cursor, GitHub Copilot, VS Code, Goose, and [[letta]].

## References

- Agent Skills — Specification. agentskills.io, 2025-12-18. <https://agentskills.io/specification>
- Documentation index: <https://agentskills.io/llms.txt>
- Source repository / governance: GitHub `agentskills/agentskills`.
- Reference validator: `skills-ref validate ./my-skill`.
