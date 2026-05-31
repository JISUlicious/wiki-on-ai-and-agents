---
title: "OpenAI Codex Agent Skills"
type: source
created: 2026-05-29
updated: 2026-05-29
sources:
  - openai-codex-skills-2025.md
authors:
  - "[[openai]]"
year: 2025
introduces:
  - "[[agent-skills]]"
tags:
  - 2025
status: complete
importance: high
---

# OpenAI Codex Agent Skills

**Source**: `sources/openai-codex-skills-2025.md` (captured from static HTML)
**URL**: https://developers.openai.com/codex/skills
**Publisher**: [[openai|OpenAI]]
**Date**: late 2025 (experimental rollout)

## Summary

OpenAI Codex ships **"Agent Skills" built on a `SKILL.md` file** — a near-exact analog of [[anthropic|Anthropic]]'s [[agent-skills|Agent Skills]] model. This is the headline significance of the source: it is the strongest available evidence that the [[skill-md-format|SKILL.md format]] is converging into a genuine **cross-vendor interop standard** rather than an Anthropic-specific convention. The page itself states that Codex skills "build on the open agent skills standard," and the design choices line up point-for-point with Anthropic's: a skill is a **directory containing a required `SKILL.md`** (with `name` + `description`) plus optional `scripts/`, `references/`, and `assets/`; capability is surfaced through **on-demand progressive disclosure** rather than upfront tool declaration; and executable scripts/resources travel alongside the instructions in the same bundle. OpenAI describes `SKILL.md` as "a playbook Codex can follow" — the same instruction-layer framing Anthropic uses.

The divergences between the two implementations are essentially cosmetic and ecosystem-level, not architectural. Codex reads skills from `.agents/skills` (vs. Anthropic's `.claude/skills`), uses an optional `agents/openai.yaml` file for UI metadata / invocation policy / tool dependencies (vs. Anthropic's reliance on plugin marketplaces and the `/v1/skills` API), and ties skills into the **`AGENTS.md` ecosystem** and Codex's plugin distribution unit. The vendor-neutral `.agents/skills` path is itself telling — OpenAI deliberately avoided a Codex-branded directory, mirroring the way the `AGENTS.md` convention was positioned as cross-tool. None of these touch the core contract: directory + `SKILL.md` + `name`/`description` metadata + lazy loading + bundled scripts.

Taken together with [[anthropic-agent-skills-2025|Anthropic's Agent Skills]] and the open specification at agentskills.io, this source validates the **agentskills.io open standard as a real interoperability format** with at least two independent major-vendor implementations. Two of the largest agent vendors arriving at the same file format, metadata schema, and progressive-disclosure mechanic — independently and within roughly a month of each other — is the kind of convergence that distinguishes a durable standard from a single lab's house style. For the wiki, it extends the "all documented agents read SKILL.md" observation from the Anthropic-adjacent ecosystem ([[claude-code]], [[opencode]], [[pi-mono]], [[hermes-agent]], [[openclaw]]) to a direct competitor's flagship coding agent.

## Key Points

- **`SKILL.md` is required**: a skill is a directory whose only mandatory file is `SKILL.md`, which must declare `name` and `description`; `scripts/`, `references/`, `assets/`, and `agents/openai.yaml` are optional. Framed as "a playbook Codex can follow."
- **Four discovery scopes** (symlinks followed, target resolved):
  - **REPO** — `.agents/skills`, scanned in every directory from the current working directory up to the repository root (`$CWD/.agents/skills`, parent folders, `$REPO_ROOT/.agents/skills`). Same-named skills are not merged; both can appear in selectors.
  - **USER** — `$HOME/.agents/skills` (personal skills, any repo).
  - **ADMIN** — `/etc/codex/skills` (machine/container-wide, for SDK scripts and default admin skills).
  - **SYSTEM** — bundled with Codex by OpenAI (e.g., `skill-creator`, `plan`).
- **Progressive disclosure with a hard budget**: Codex preloads only each skill's `name`, `description`, and file path. This initial list is capped at **~2% of the context window, or 8,000 characters** when the window is unknown; over-budget skill sets get shortened descriptions first, then omissions (with a warning). The **full `SKILL.md` loads only when a skill is selected**.
- **Two invocation modes**: **explicit** (`/skills` command or `$`-mention in the prompt) and **implicit** (Codex matches the task to the skill `description` — so descriptions should front-load trigger words and scope).
- **`agents/openai.yaml`** (optional) sets UI metadata (`display_name`, icons, `brand_color`, `default_prompt`), invocation policy (`allow_implicit_invocation`, default `true`), and tool/MCP dependencies (e.g., declaring an [[mcp|MCP]] server the skill needs).
- **Skills vs. plugins**: skills are the authoring format for reusable workflows; **plugins** are the installable distribution unit (one or more skills, optionally with app mappings + MCP config). `$skill-creator` scaffolds new skills; `$skill-installer` adds curated ones locally.
- **Availability**: Codex CLI, IDE extension, and Codex app. Skill changes auto-detected; `~/.codex/config.toml` `[[skills.config]]` entries can disable a skill without deleting it.
- **Convergent with Anthropic**: directory + `SKILL.md` + `name`/`description` + progressive disclosure + bundled scripts — the same contract, independently implemented.

## Comparison: Anthropic vs. OpenAI Codex Agent Skills

| Dimension | [[anthropic-agent-skills-2025\|Anthropic Agent Skills]] | OpenAI Codex Agent Skills (this source) |
|---|---|---|
| Format file | `SKILL.md` (required, `name` + `description`) | `SKILL.md` (required, `name` + `description`) |
| Repo skills path | `.claude/skills` | `.agents/skills` (vendor-neutral) |
| User skills path | `~/.claude/skills` | `$HOME/.agents/skills` |
| Discovery scopes | platform / Code (`~/.claude/skills`) / plugin marketplace | repo / user / admin (`/etc/codex/skills`) / system (bundled) |
| Progressive disclosure | three levels (L1 metadata → L2 body → L3 files) | metadata preloaded (~2% / 8,000-char cap) → full `SKILL.md` on selection |
| Optional metadata | plugin/marketplace manifest; `/v1/skills` API | `agents/openai.yaml` (UI, invocation policy, deps) |
| Invocation | auto-invocation + explicit | implicit (description match) + explicit (`/skills`, `$`) |
| Distribution unit | plugins / `anthropics/skills` marketplace | plugins; ties into `AGENTS.md` ecosystem |
| Open standard | published at agentskills.io (Dec 2025) | "build on the open agent skills standard" |

The two implementations are **format-compatible at the core** (`SKILL.md` + `name`/`description` + on-demand loading); differences are confined to file paths, metadata side-files, and distribution plumbing.

## Entities Mentioned

- [[openai]] — publisher; ships Codex Agent Skills
- [[anthropic]] — originator of the SKILL.md format, referenced here for comparison
- [[claude-code]] — Anthropic's developer surface that pioneered the format; the direct counterpart to Codex

## Concepts Discussed

- [[agent-skills]] — the canonical capability-packaging concept (introduced for Anthropic; this source is the OpenAI implementation)
- [[skill-md-format]] — the file format now shown to be cross-vendor
- [[mcp]] — declarable as a skill dependency via `agents/openai.yaml`

## References

_Original source: `sources/openai-codex-skills-2025.md`_
_Documentation: https://developers.openai.com/codex/skills_
_Cross-link: [[anthropic-agent-skills-2025]] — the convergent Anthropic announcement_
_Cross-link: [[agentskills-io-spec-2025]] — the open standard both implementations claim to build on_
_See also: github.com/openai/skills and the agent skills specification_
