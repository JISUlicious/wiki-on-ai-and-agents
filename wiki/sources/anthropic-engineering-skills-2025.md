---
title: "Equipping Agents for the Real World with Agent Skills"
type: source
created: 2026-05-29
updated: 2026-05-29
sources:
  - anthropic-engineering-skills-2025.md
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

# Equipping Agents for the Real World with Agent Skills

**Source**: `sources/anthropic-engineering-skills-2025.md` (captured from static HTML — some original formatting, diagrams, and code listings are lost in the capture)
**URL**: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
**Publisher**: [[anthropic|Anthropic]] (Engineering blog)
**Date**: 2025-10-16
**Authors**: Barry Zhang, Keith Lazuka, Mahesh Murag

## Summary

This is the engineering deep-dive that accompanies [[anthropic|Anthropic]]'s [[agent-skills|Agent Skills]] launch — the technical companion to the product announcement filed as [[anthropic-agent-skills-2025]]. Where the announcement frames Skills as composable, portable capability bundles, this piece explains the *mechanics*: how the [[skill-md-format|SKILL.md]] format, progressive disclosure, and code execution combine to let a general-purpose agent specialize at runtime without paying for that specialization up front in context. The framing analogy throughout is that building a skill is "like putting together an onboarding guide for a new hire" — capturing procedural knowledge and organizational context into reusable files and folders.

The core technical claim is **progressive disclosure across three (or more) levels**, anchored to the agent's filesystem. **L1** — at startup, only the `name` and `description` of every installed skill are preloaded into the system prompt, giving Claude just enough to know *when* a skill applies. **L2** — when Claude judges a skill relevant, it reads the full SKILL.md body into context (in the worked PDF example, by invoking a Bash tool to read `pdf/SKILL.md`). **L3 and beyond** — SKILL.md can reference additional bundled files by name (e.g. `reference.md`, `forms.md`), which Claude navigates and loads only as needed. Because Claude has a filesystem and code-execution tools, it never needs the entire skill resident in its context window; the article states that the amount of context that can be bundled into a skill is therefore **effectively unbounded**.

The second mechanic is the **code-execution model**, which goes materially deeper than the announcement. Skills can bundle scripts that Claude runs as deterministic tools at its discretion. Critically, Claude can run a script "without loading either the script or the [input] into context" — in the PDF example, a Python script extracts form fields from a PDF, and **only the script's output consumes tokens**, not the script source or the PDF itself. This makes operations like sorting or form parsing both cheaper (no token-by-token generation) and reliably repeatable (deterministic). The article also advances a "**code as both executable tool and documentation**" framing: a bundled script can be *run* by Claude or *read* by Claude as reference, and skill authors should make clear which is intended. This is a philosophy/mechanics piece — there are **no benchmark numbers and no quantitative case studies**; it argues from design principles and one running PDF-skill example, then offers authoring guidance.

## Key Points

- **Three-level (L1/L2/L3+) progressive disclosure**, the core design principle:
  - **L1** — `name` + `description` of every installed skill preloaded into the system prompt at startup; enough to decide *when* to trigger, without loading the skill body.
  - **L2** — the full `SKILL.md` body is read into context only when Claude judges the skill relevant to the current task (triggered via the system prompt, executed by reading the file with a Bash tool).
  - **L3 and beyond** — additional files bundled in the skill directory and referenced by name from SKILL.md (e.g. `reference.md`, `forms.md`), navigated and loaded by Claude only as needed. Keeping rarely-co-used paths in separate files reduces token usage.
- **Context bundled into a skill is effectively unbounded** — because the agent has a filesystem and code execution, it loads only the slice of a skill it needs, so a skill can carry far more material than would fit in a single context window.
- **Code runs without entering context** — Claude can execute a bundled script via Bash without loading the script *or* its inputs into the context window; only the script's *output* consumes tokens.
- **Determinism + efficiency** — certain operations (the article cites sorting, and form-field extraction in the PDF example) are cheaper and more reliable as code than as token generation; code gives consistent, repeatable workflows.
- **Code as dual artifact** — a bundled script can serve simultaneously as an executable tool and as embedded documentation; authors should make explicit whether Claude is meant to run a script or read it as reference.
- **Worked example**: the PDF document-editing skill. SKILL.md references `reference.md` and `forms.md`; the form-filling instructions live in `forms.md` so the core stays lean, trusting Claude to read it only when filling a form. A bundled Python script extracts form fields.
- **Authoring guidance**: start from evaluation (find capability gaps on representative tasks, build incrementally); structure for scale (split unwieldy SKILL.md into referenced files); think from Claude's perspective (monitor real trajectories, watch for overreliance, tune `name`/`description` since they drive triggering); iterate *with* Claude (have Claude capture successful approaches and self-reflect on failures into reusable skill context).
- **Security**: malicious skills can introduce vulnerabilities or direct Claude to exfiltrate data; install only from trusted sources, audit bundled files (especially code dependencies and instructions that reach untrusted external network sources) before use. See [[indirect-prompt-injection]] / [[prompt-injection]] for the broader threat class.
- **Relationship to [[mcp|MCP]]**: the article positions Skills as complementary to [[mcp|Model Context Protocol]] servers — Skills can teach agents complex workflows that *involve* external tools/software exposed via MCP.
- **Forward look**: enabling agents to create, edit, and evaluate Skills on their own — codifying their own behavior patterns into reusable capabilities (a [[self-improving-agent|self-improvement]] direction).
- **How this deepens the announcement**: [[anthropic-agent-skills-2025]] establishes *what* Skills are and *where* they run (claude.ai, Claude Code, Agent SDK, Developer Platform); this engineering post goes deeper on *how* — filesystem-navigated progressive disclosure as the lever for unbounded context, and the "code as both tool and doc" execution model where running a script costs only its output in tokens.

## Entities Mentioned

- [[anthropic]] — author and originator of Agent Skills
- [[claude-code]] — cited as an agent that already does local code execution and filesystem work, the substrate Skills build on

## Concepts Discussed

- [[agent-skills]] — the concept this source introduces and explains mechanically
- [[skill-md-format]] — the SKILL.md file format (YAML frontmatter + markdown body + referenced bundled files)
- [[code-as-harness]] — code as deterministic tool, run without loading source/inputs into context; the "code as both tool and documentation" framing
- [[mcp]] — Model Context Protocol; positioned as complementary to Skills
- [[agentic-context-engineering]] — progressive disclosure / context assembly: managing what enters the context window and when (the source's `context-assembly` theme)
- [[memory-management]] — filesystem-as-context navigation; loading only the needed slice rather than holding everything resident

## Notable Quotes

> "the amount of context that can be bundled into a skill is effectively unbounded."

> "Claude can run this script without loading either the script or the PDF into context."

> "code can serve as both executable tools and as documentation."

> "Building a skill for an agent is like putting together an onboarding guide for a new hire."

_The quotes above are reconstructed from a static-HTML capture; wording is verified against the captured text but punctuation/formatting may differ slightly from the rendered original._

## References

- Companion announcement (the product launch this piece elaborates): [[anthropic-agent-skills-2025]]
- Original source: `sources/anthropic-engineering-skills-2025.md`
- Live URL: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
