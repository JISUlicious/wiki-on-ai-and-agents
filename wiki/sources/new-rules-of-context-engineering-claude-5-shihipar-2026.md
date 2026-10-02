---
title: "The new rules of context engineering for Claude 5 generation models"
type: source
created: 2026-10-02
updated: 2026-10-02
sources:
  - new-rules-of-context-engineering-claude-5-shihipar-2026.md
authors:
  - "[[thariq-shihipar]]"
year: 2026
venue: "claude.dev (Anthropic developer blog)"
publisher: Anthropic
status: complete
importance: high
tags:
  - 2026
---

# The new rules of context engineering for Claude 5 generation models

**Source**: `sources/new-rules-of-context-engineering-claude-5-shihipar-2026.md` (official [[anthropic|Anthropic]] developer blog)
**Author**: [[thariq-shihipar|Thariq Shihipar]] (member of technical staff, Anthropic)
**Date**: 2026-07-24
**URL**: https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/

## Summary

Anthropic says it "removed over 80% of Claude Code's system prompt for models like Claude Opus 5 and Claude Fable 5 with no measurable loss on our coding evaluations". The post draws context-engineering lessons from that change. Its argument is that older Claude models needed hard rules, examples, repetition and everything up front, whereas Claude 5 generation models do better with judgment, well-designed interfaces and [[agent-skills|progressive disclosure]]. The 80% figure is **Anthropic-reported**, and the evals behind it are not described. Everything else is **advice** drawn from Anthropic's own transcripts.

## Key points

- **Over-constraint**: internal transcripts showed conflicting instructions in a single request, such as "leave documentation as appropriate" next to "DO NOT add comments". Resolving that kind of conflict costs the model deliberation.
- **Then/now pairs (advice)**:
  - **Rules → judgment**: the old rule "default to writing no comments…" became "match its comment density, naming, and idiom".
  - **Examples → interface design**: with the newest models, examples "constrain them to a certain exploration space". Expressive tool parameters (for example a status enum) do the work instead.
  - **Everything up front → progressive disclosure**: verification and code review moved into skills. Deferred tools are loaded through ToolSearch.
  - **Repetition → simple tool descriptions**: usage instructions go in the tool description, not duplicated in the system prompt.
  - **`CLAUDE.md` memory via the `#` hotkey → auto-memory.**
  - **Simple specs → rich references**: HTML artifacts, test suites, code to port, and rubrics checked by verifier agents through [[dynamic-workflows]].
- **Applying it (advice)**:
  - The system prompt is product context.
  - `CLAUDE.md` should be light and spend its tokens on "gotchas", not "the obvious".
  - Skills should be lightweight guides, split into many files.
  - References should be code where possible: "a HTML mockup… will generally produce better results than a description of the design or a screenshot".
- **Tooling**: `claude doctor` / `/doctor` "rightsize[s]" skills and `CLAUDE.md` files.

## Concepts & entities

- [[context-assembly]]: the post is a practitioner guide to the L2 assembly layer (system prompt, `CLAUDE.md`, skills, memory, references).
- [[agent-skills]] · [[skill-md-format]]: progressive disclosure as the main mechanism. [[memory-management]]: auto-memory.
- [[agentic-context-engineering]]: see the contradiction note below.
- [[why-does-claude-md-keep-growing-chakrabarti-2026]]: measured `CLAUDE.md` growth, the problem `/doctor` targets.
- [[dynamic-workflows]] · [[claude-code-dynamic-workflows-harness-2026]] (same author): rubric verifiers and a tree of files.
- [[tool-use]] · [[claude-code]] · [[claude]] · [[anthropic]] · [[thariq-shihipar]].

> [!warning] Contradiction: comprehensive playbooks vs. pruned context
> [[agentic-context-engineering|ACE]] prescribes "comprehensive playbooks, not concise summaries" and warns that rewriting collapses context (18,282 → 122 tokens, 66.7% → 57.1%). This post reports deleting more than 80% of a production system prompt "with no measurable loss" and calls the central-repository `CLAUDE.md` "a common myth". The two may be reconcilable: ACE's playbooks hold task-specific lessons, while Anthropic's cuts removed generic rules and moved detail into skills that load on demand.

## What it does not show

- The coding evaluations, the size of the prompt before and after, and any per-task variance are all undisclosed. "No measurable loss" is the only result given.
- The advice is scoped to Claude 5 generation models and gives no evidence that it transfers to other model families.

## References

- https://claude.dev/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models/
- [[claude-code-dynamic-workflows-harness-2026]], which this post links for "a tree of files".
- _Original source: `sources/new-rules-of-context-engineering-claude-5-shihipar-2026.md`_
