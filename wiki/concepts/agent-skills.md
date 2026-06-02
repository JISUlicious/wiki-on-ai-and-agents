---
title: Agent Skills (Anthropic)
type: concept
created: 2026-05-16
updated: 2026-05-31
sources:
  - anthropic-agent-skills-2025.md
  - anthropic-engineering-skills-2025.md
  - agentskills-io-spec-2025.md
  - openai-codex-skills-2025.md
  - agent-skills-survey-xu-2026.md
status: complete
importance: high
tags:
  - 2025
---

# Agent Skills (Anthropic)

**Agent Skills** are [[anthropic|Anthropic]]'s architecture for packaging and distributing specialized capabilities for [[claude|Claude]] as portable, composable, versioned bundles of instructions, scripts, and resources. Announced October 16, 2025 and published as an open standard at **agentskills.io** in December 2025, a Skill is a directory whose entry point is a [[skill-md-format|SKILL.md]] file — YAML frontmatter (minimally `name` and `description`) plus a markdown body of workflow instructions, with optional executable code, templates, and reference files referenced from the body.

## Three-level progressive disclosure

The defining architectural choice. Context is loaded lazily across three levels so that a Claude instance can have thousands of skills installed without paying the context-window cost up front.

- **Level 1 — Metadata in system prompt**: at startup, only `name` + `description` of each installed skill is preloaded into Claude's system prompt. This is what Claude uses during planning to decide *whether* a skill applies.
- **Level 2 — Body loaded on demand**: once Claude judges a skill relevant to the task at hand, the full SKILL.md body (workflow steps, examples, constraints) is loaded into context.
- **Level 3 — Supporting files referenced from body**: scripts, templates, and reference docs in the skill directory are loaded only when the body's instructions explicitly reference them mid-execution.

The cumulative effect: planning sees a thin index; execution unfolds the relevant subtree on demand. This is the principle that makes the format scale to organization-wide skill catalogs.

### Filesystem navigation + code execution (engineering deep-dive)

The [[anthropic-engineering-skills-2025|engineering companion]] to the announcement adds two mechanics absent from the headline framing:

- **L3+ is filesystem navigation, not a fixed third tier.** Because Claude can read bundled files by name on demand, the context a skill can reference is *effectively unbounded* — a skill directory is a tree Claude walks as needed, like an onboarding guide for a new hire, rather than a flat third level.
- **Bundled code runs without entering context.** A skill's scripts execute via bash; neither the script source nor its inputs are loaded into the context window — **only the output consumes tokens**. This gives code a dual role: a deterministic tool *and* embedded documentation. (Token budgets, per the spec: metadata ≈100 tokens at L1; SKILL.md body recommended <5000 tokens at L2.)

## The SKILL.md format

See [[skill-md-format]] for the cross-agent file-format specification. The minimal shape is:

```markdown
---
name: my-skill
description: One-line description of what this skill does
---

# Skill body — instructions the LLM follows when this skill is invoked.
```

Skills can include multiple files in their directory; reference content can be split out when the SKILL.md becomes unwieldy. The body may reference executable code that the runtime executes (in [[claude-code|Claude Code]]'s case, locally; in the API's case, via the Code Execution Tool sandbox).

> [!warning] Frontmatter is not just name + description
> The formal [[agentskills-io-spec-2025|agentskills.io spec]] (published 2025-12-18) defines **six** frontmatter fields — `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools` — with strict naming rules (`name` 1–64 chars, **must match the parent directory name**, no consecutive hyphens; `description` ≤1024 chars). [[claude-code|Claude Code]] adds ~15 more runtime-specific fields. See [[skill-md-format]] for the full schema table.

## Cross-platform availability

The same Skill works across Anthropic's surfaces — Anthropic's core portability promise:

- **Claude apps (claude.ai)** — Pro/Max/Team/Enterprise users. Three categories: platform-provided (e.g. document generation), example (customizable starters), and user-authored. A built-in **skill-creator** skill scaffolds new ones interactively.
- **[[claude-code|Claude Code]]** — plugins from the `anthropics/skills` marketplace, or manual install at `~/.claude/skills/`. Version-control-friendly: skills go in git like any other source asset.
- **Claude Developer Platform (API)** — skills attach via `container.skills[]` (**max 8 per request**, each `{type, skill_id, version}`); requires the code-execution tool plus beta headers (`code-execution-2025-08-25, skills-2025-10-02, files-api-2025-04-14`). Full `/v1/skills` CRUD + `/versions` for programmatic version management (upload limit <30 MB; not ZDR-eligible; handle the `pause_turn` stop reason for long operations).
- **Claude Agent SDK** — same Agent Skills support for building custom agents on Anthropic's stack.

## Four properties

Anthropic frames Skills around four properties differentiating them from naive tool integration:

| Property | Meaning |
|---|---|
| **Composability** | Skills stack. Claude identifies which apply and coordinates them — no manual selection. |
| **Portability** | Same format across apps, [[claude-code|Code]], and API. Build once, run anywhere. |
| **Efficiency** | Load only what's needed, when it's needed (progressive disclosure). |
| **Power** | Skills can include executable code — deterministic execution beats token generation for many tasks. |

## Comparison with function calling and MCP

Skills are positioned against three adjacent approaches:

- **vs. [[function-calling|function calling]] / [[tool-use|tool use]]**: traditional tools require *upfront declaration* of every tool in the system prompt; Skills are *lazily loaded* and selected by relevance. Function-calling answers "what tools exist?"; Skills answer "what knowledge + workflow + tools does this task need, loaded just-in-time?"
- **vs. system prompts**: system prompts are static text; Skills are versioned artifacts you can deploy, update, audit, and revoke.
- **vs. [[mcp|MCP]]**: [[mcp|MCP]] standardizes the *wire protocol* for connecting agents to external tools (an N+M client-server protocol). Skills standardize *content portability* — the file format for instruction + code bundles. The two are complementary: an MCP server exposes tools an agent can call; a Skill tells the agent *how to use* tools (whether MCP-served or built-in) for a particular workflow. In the [[agent-three-layer-model|three-layer agent model]], Skills are an instruction-layer asset riding on top of L2 action surfaces.

## Security and governance

Because Skills can ship executable code (L3), Anthropic emphasizes a trust model:

- **Org-wide enable gate**: Team/Enterprise admins must explicitly enable Skills before individual users can access them.
- **Trusted sources warning**: Anthropic's own guidance — *"This feature gives Claude access to execute code. While powerful, it means being mindful about which skills you use—stick to trusted sources to keep your data safe."*
- **December 2025 updates**: organization-wide management for skills, a partner-built skills directory — moving toward marketplace + enterprise governance.
- **Decoupling reasoning from execution** also yields interpretability (which skill ran?), auditability (skill invocations create trails), and independent specialization (skills are versioned + improved separately from Claude itself).

## Open standard

**2025-12-18** publication of Agent Skills at **[[agentskills-io-spec-2025|agentskills.io]]** as an open standard for cross-platform portability, maintained by Anthropic with open ecosystem contributions (GitHub: `agentskills/agentskills`) and a reference validator (`skills-ref validate`). This parallels [[anthropic|Anthropic]]'s donation of [[mcp|MCP]] to the Linux Foundation a month earlier — both moves signal intent to establish these formats as community infrastructure rather than proprietary lock-in.

## Academic grounding

The concept now has dedicated academic treatment. [[agent-skills-survey-xu-2026|Xu & Yan 2026]] (Zhejiang University) is the **first survey dedicated to agent skills**, defining a skill as "composable packages of instructions, code, and resources that agents load on demand" — congruent with Anthropic's operational definition but generalized into a paradigm-level abstraction positioned beyond prompt engineering and tool use. The survey organizes the field across **architecture / acquisition / deployment / security** and traces the lineage through [[voyager]]'s executable [[skill-library]], the program-synthesis [[library-learning]] thread ([[dreamcoder]] → [[lilo-grand-2023|LILO]]), the self-authored [[tool-creation]] thread ([[llm-as-tool-makers-cai-2023|LATM]], [[craft-yuan-2023|CRAFT]]), and the natural-language [[experiential-learning]] counterpoint ([[expel-zhao-2023|ExpeL]]). See [[skill-acquisition]] for the acquisition taxonomy.

## Cross-vendor convergence

The SKILL.md format is converging into a genuine cross-vendor standard, not an Anthropic-only artifact:

- **[[openai-codex-skills-2025|OpenAI Codex]]** independently shipped `SKILL.md` — same directory + name/description + progressive-disclosure model (uses `.agents/skills`, an `agents/openai.yaml` metadata file, and ties to the AGENTS.md ecosystem).
- **Google Gemini CLI Extensions** bundle "Agent Skills" with explicit on-demand activation.
- 40+ adopters listed at agentskills.io (Cursor, GitHub Copilot, VS Code, Goose, [[letta|Letta]], …).
- The hyperscaler enterprise stacks keep proprietary equivalents (Microsoft Declarative Agents, AWS Bedrock Action Groups, OpenAI GPTs, Google Gems) that map to the same conceptual slots without sharing the file format.

See **[[agent-skills-cross-vendor]]** for the full comparison.

## Cross-agent adoption (2026 status)

By early 2026, the [[skill-md-format|SKILL.md format]] is read by all five documented agents in this wiki — though not all explicitly implement Anthropic's three-level progressive disclosure:

- [[claude-code]] — origin runtime; full L1/L2/L3 disclosure.
- [[opencode]] — explicit Claude-Code compatibility; walks `~/.claude/skills/` and `.agents/skills/`.
- [[pi-mono]] — walks `~/.pi/` and `~/.agents/`; refers to "Agent Skills v1" interop.
- [[hermes-agent]] — Anthropic-inspired plus agentskills.io compatibility; bundled + optional skills.
- [[openclaw]] — inherits from pi.

The convergence happened organically rather than via formal standardization, because the format is simple enough that adoption costs nothing and the network effect favors the format with the largest skill ecosystem.

## References

- [[anthropic-agent-skills-2025]] — the Oct 2025 announcement source
- [[anthropic-engineering-skills-2025]] — the engineering deep-dive (filesystem mechanics, code-execution model)
- [[agentskills-io-spec-2025]] — the formal open standard (six-field frontmatter schema)
- [[openai-codex-skills-2025]] — OpenAI's convergent SKILL.md adoption
- [[agent-skills-survey-xu-2026]] — the academic survey (Xu & Yan, Zhejiang University)
- [[agent-skills-cross-vendor]] — cross-vendor capability-packaging comparison
- [[skill-md-format]] — file-format details and cross-agent adoption notes
- [[skill-acquisition]] — how agents acquire skills
- [[skill-optimization]] — automatically improving a skill artifact (SkillOpt, SkillMOO, SkillReducer, Skill1, SAGE)
- [[skill-evaluation]] — measuring skill usefulness in realistic settings
- [[mcp]] — complementary tool-integration protocol
- [[function-calling]] — the prior pattern Skills extend
- [[tool-use]] — broader category
- [[agent-three-layer-model]] — where Skills fit architecturally
- Announcement: https://claude.com/blog/skills
- Engineering: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
- Open standard: https://agentskills.io
