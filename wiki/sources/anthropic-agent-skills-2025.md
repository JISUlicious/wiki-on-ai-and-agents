---
title: "Introducing Agent Skills"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - anthropic-agent-skills-2025.md
authors:
  - "[[anthropic]]"
year: 2025
introduces:
  - "[[agent-skills]]"
tags:
  - 2025
status: complete
importance: high
---

# Introducing Agent Skills

**Source**: `sources/anthropic-agent-skills-2025.md` (fetched from https://claude.com/blog/skills)
**Publisher**: [[anthropic|Anthropic]]
**Date**: 2025-10-16 (initial announcement); reflects evolution through December 2025 (open-standard publication, enterprise governance)

## Summary

[[agent-skills|Agent Skills]] are [[anthropic|Anthropic]]'s mechanism for packaging and distributing specialized knowledge and executable capabilities for [[claude]] across multiple platforms. Announced October 16, 2025, Skills are defined as "folders that include instructions, scripts, and resources that Claude can load when needed." A skill is a directory containing a [[skill-md-format|SKILL.md]] file (with YAML frontmatter declaring `name` and `description`) plus optional supporting files — executable scripts, templates, reference content. The same format works across [[claude]] consumer apps (claude.ai), [[claude-code|Claude Code]], the Messages API, and the Claude Agent SDK.

The architectural innovation is **progressive disclosure across three levels**: (L1) at startup, only `name` + `description` of each installed skill is preloaded into the system prompt; (L2) when Claude judges a skill relevant, the full SKILL.md body loads into context; (L3) supporting files (code, templates) load only when the body references them during execution. This keeps the planning context light while making rich capability available on demand. Anthropic frames Skills around four properties: **composability** (skills stack and Claude coordinates them automatically), **portability** (one format across apps, Code, and API), **efficiency** (lazy loading), and **power** (executable code where deterministic execution beats token generation).

Positioning is explicit: Skills differ from traditional [[function-calling|function calling]] / [[tool-use|tool use]] (which require upfront declaration of all tools) by enabling lazy, relevance-based loading; from system prompts (static) by being versioned, deployable, and governed; and from [[mcp|MCP]] (which standardizes inter-process protocol for tool connections) by emphasizing content portability across deployments rather than wire-protocol standardization. The two are complementary: Skills package *how to do something*; MCP exposes *what to connect to*. In [[agent-three-layer-model|the three-layer agent model]], Skills are an instruction-layer artifact riding on top of L2 action surfaces.

A significant December 2025 development is the publication of Agent Skills as an **open standard at agentskills.io**, plus enterprise organization-wide management and a partner-skills directory. The open-standard move parallels [[anthropic|Anthropic]]'s donation of [[mcp|MCP]] to the Linux Foundation a month earlier — both signal intent to establish these formats as community infrastructure rather than proprietary lock-in. By 2026, all five documented agents in the wiki ([[claude-code]], [[opencode]], [[pi-mono]], [[hermes-agent]], [[openclaw]]) read the [[skill-md-format|SKILL.md format]], making it the de facto cross-agent interop standard.

## Key Points

- **Announced**: 2025-10-16 by [[anthropic|Anthropic]].
- **Definition**: skills are folders bundling instructions, scripts, and resources that [[claude|Claude]] loads when needed.
- **SKILL.md is the entry point**: YAML frontmatter with `name` + `description`; markdown body holds workflow instructions; auxiliary files referenced from the body.
- **Three-level progressive disclosure**:
  - L1 — metadata (name + description) preloaded into the system prompt at startup
  - L2 — full SKILL.md body loaded on demand when Claude judges the skill relevant
  - L3 — supporting files (code, templates, configs) loaded only when the body references them
- **Four properties**: composability, portability, efficiency, power.
- **Cross-platform availability**:
  - **Claude apps (claude.ai)** — Pro/Max/Team/Enterprise; platform-provided, example, and user-authored skills; auto-invocation; a "skill-creator" skill scaffolds new ones.
  - **[[claude-code|Claude Code]]** — plugins from `anthropics/skills` marketplace, or manual install at `~/.claude/skills`. Git-versionable.
  - **Claude Developer Platform (API)** — Messages API integration; `/v1/skills` endpoint for programmatic version management; requires Code Execution Tool beta.
  - **Claude Agent SDK** — same skills support for custom agents.
- **Vs. [[function-calling|function calling]] / [[tool-use|tool use]]**: traditional tools require upfront declaration; skills are lazily loaded and relevance-selected.
- **Vs. system prompts**: skills are versioned, deployable, governed; system prompts are static.
- **Vs. [[mcp|MCP]]**: MCP standardizes protocol for inter-process tool connections; Skills standardize *content portability* of instruction + code bundles.
- **Governance**: Team/Enterprise admins must enable Skills org-wide before users access them — reflects that skills can execute code.
- **Open standard (Dec 2025)**: published at **agentskills.io** for cross-platform portability.
- **Partner use cases**: Box (document generation per org standards), Canva (agent customization), Notion (action automation), Rakuten ("a day → an hour" for management accounting workflows).
- **Decoupling thinking from doing** enables interpretability, auditability, governance, and independent specialization of capabilities.

## Entities Mentioned

- [[anthropic]] — originator
- [[claude]] — the consuming model
- [[claude-code]] — primary developer surface; ships with skills plugin marketplace
- Box, Canva, Notion, Rakuten — launch partner orgs with documented use cases

## Concepts Discussed

- [[agent-skills]] — the canonical concept introduced here
- [[skill-md-format]] — the file format
- [[mcp]] — complementary tool-integration protocol
- [[function-calling]] — the prior approach that Skills extend
- [[tool-use]] — broader pattern Skills participate in
- [[agent-three-layer-model]] — Skills as an instruction-layer artifact riding on the action surface

## Notable Quotes

> "Skills stack together. Claude automatically identifies which skills are needed and coordinates their use."

> "Skills use the same format everywhere. Build once, use across Claude apps, Claude Code, and API."

> "Skills can include executable code for tasks where traditional programming is more reliable than token generation."

> "This feature gives Claude access to execute code. While powerful, it means being mindful about which skills you use—stick to trusted sources to keep your data safe."

## References

_Original source: `sources/anthropic-agent-skills-2025.md`_
_Announcement: https://claude.com/blog/skills_
_Open standard: https://agentskills.io_
