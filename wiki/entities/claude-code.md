---
title: Claude Code
type: entity
created: 2026-05-14
updated: 2026-10-02
sources:
  - claude-code.md
  - comparison.md
  - claude-code-dynamic-workflows-2026.md
  - claude-code-dynamic-workflows-harness-2026.md
  - claude-effort-and-ultracode-docs-2026.md
  - anthropic-threat-intelligence-report-september-2026.md
family: agent-product
parameters: "n/a (powered by Claude)"
release_date: 2024-02-29
status: complete
importance: high
tags:
  - product
---

# Claude Code

Claude Code is [[anthropic|Anthropic]]'s official terminal-based coding agent. Distributed as the `@anthropic/claude-code` npm package. Powered by [[claude|Claude]] models (Opus / Sonnet / Haiku). The CLI maintaining this very wiki is Claude Code.

## Architecture (per the 2026 sourcemap-snapshot writeup)

- **TypeScript / React-Ink TUI**.
- **~5,000-line REPL orchestrator** (`main.tsx`), **~1,700-line turn state machine** (`query.ts`).
- **~40 built-in tools** with a unified Zod-schema contract: Bash, Read, Edit, Write, Glob, Grep, Task (sub-agent dispatch), MCP, WebFetch, WebSearch, NotebookEdit, AskUserQuestion, ExitPlanMode, and more.
- **Sub-agents are first-class** via the `Task` tool; built-in subagent types include Explore, Plan, code-reviewer, claude-code-guide, etc. Subagents can run in worktree-isolated or remote-isolated modes.
- **Memory subsystem**: per-project file-based memory at `~/.claude/projects/{slug}/memory/`, with a background "auto-dream" consolidator that periodically distills memory into compact reusable forms.
- **Permissions system**: layered. Rules + classifier + plan-mode + hooks. Separates *whether* a call is allowed from *what runtime capabilities* it has.
- **Sandbox**: OS-level via `@anthropic-ai/sandbox-runtime` adapter (FS/network rules).
- **MCP**: first-class. Tools can be deferred and loaded on demand via `ToolSearch`.
- **Hooks**: pre-tool, post-tool, on-stop, on-error. Configurable in `settings.json`.
- **CLAUDE.md**: nested in project tree, auto-loaded at startup, holds domain-specific schema (the wiki you are reading lives in such a CLAUDE.md ecosystem).

## Dynamic workflows (2026)

As of May–June 2026, Claude Code can **write and orchestrate its own multi-agent harness on the fly** via [[dynamic-workflows]]: Claude authors a JavaScript orchestration script that spawns tens-to-hundreds of parallel subagents (reusing the `Task`-tool subagent + worktree machinery above), executed by a background runtime while the session stays responsive. Enabled by [[claude-opus-4-8|Claude Opus 4.8]]; surfaced via the [[ultracode]] setting in the `/effort` menu (which pairs `xhigh` [[reasoning-effort|effort]] with standing workflow-launch permission) and composable with `/loop` and `/goal`. (Distinct from the unofficial community [[ultrawork]] trend.) See the [[claude-code-dynamic-workflows-2026|announcement]] and [[claude-code-dynamic-workflows-harness-2026|engineering deep-dive]]. This makes the self-authored harness a first-class Claude Code capability — the [[code-as-harness]] thesis applied to Claude Code itself.

## In the cross-agent landscape

- Distinguished by **scale** (~40 tools, first-class sub-agents, layered permissions, OS-level sandbox) — most other documented agents are simpler.
- **Compatible with the broader SKILL.md ecosystem**: pi-mono, opencode, Hermes, and OpenClaw all walk the same skill directories that Claude Code does, treating SKILL.md as a de-facto interop format.
- Source code was leaked via npm sourcemap in March 2026 and was mirrored locally for the writeup.

## In misuse reporting (2026)

Anthropic's September 2026 [[anthropic-threat-intelligence-report-september-2026|threat report]] names Claude Code in several cases. A Yemen-based cell used it "in place of human software engineers" to write guidance, navigation and control software for a guided rocket. A Russia-based group used it to write and test code for an autonomous FPV drone swarm, saving directly into their project files. On the distillation side, the report says DeepSeek recognized Claude Code users by their request strings and relayed their requests to Claude Opus. See [[ai-enabled-cyber-operations]] and [[illicit-distillation]].

> [!note] Update 2026-10-02
> - **Mods.** Beyond the `settings.json` hooks described above, Claude Code now supports [[claude-code-mods|mods]] (v2.1.287): in-process hook modules shipped as plugins, which can rewrite events and draw UI. Several built-in features are themselves mods.
> - **Tool count.** This page cites "~40 built-in tools" from a source-map writeup. Anthropic's own [[seeing-like-an-agent-shihipar-2026|Seeing like an agent]] (April 2026) says "~20 tools". The difference is probably counting method or date; neither source gives a list.
> - **Prompt size.** Anthropic reports cutting over 80% of the system prompt for Claude 5-generation models ([[new-rules-of-context-engineering-claude-5-shihipar-2026]]). The harness is built around prefix caching ([[lessons-from-building-claude-code-prompt-caching-is-everything-shihipar-2026]]).

## References

- [[claude-code-agent-doc-2026]]
- [[cross-agent-comparison-doc-2026]]
- [[claude-code-dynamic-workflows-2026]] — dynamic workflows announcement
- [[claude-code-dynamic-workflows-harness-2026]] — dynamic workflows engineering deep-dive

## Related 2026 sources

- [[ai-native-sdlc]] — the lifecycle claim this tool anchors, and where the industry narrative meets the wiki's controlled measurements
- [[flash-moe-ssd-expert-streaming-woods-2026]] — a 24-hour *autoresearch* build: ~90 propose-run-read experiments, generating the Objective-C and Metal for an SSD expert-streaming engine that runs a 397B MoE on a 48 GB laptop. Also a caution — the loop's claim that 2-bit was "indistinguishable from 4-bit" was not backed by a described evaluation, and the 2-bit build broke tool calling
- **Plugin evals shipped** (product news, AI Agents Weekly 2026-09-12) — `claude plugin eval` runs a plugin's cases with and without the plugin; its use is documented on [[building-an-ai-native-sdlc-plugin]].
- [[building-with-claude-sonnet-5-5-osmani-2026]] — Osmani's Sonnet 5.5 launch guide: spec sheet, pricing, six breaking migration changes from Sonnet 5
- [[getting-the-most-out-of-opus-5-5-osmani-2026]] — Osmani on prompting Opus 5.5: define done, drop "think hard", stop rules in CLAUDE.md, subagents
- [[what-a-task-costs-on-opus-5-5-osmani-2026]] — Worked cost model for an agentic Claude Code task on Opus 5.5; what the "40% cheaper" claim actually means
- [[new-rules-of-context-engineering-claude-5-shihipar-2026]] — Shihipar: Anthropic cut 80%+ of Claude Code's system prompt for Claude 5 models with no measured loss
- [[getting-started-with-claude-code-mods-osmani-2026]] — Osmani's tutorial on Claude Code mods: in-process JS/TS hook modules shipped as plugins
- [[using-claude-code-spending-your-effort-shihipar-2026]] — Shihipar: on Terminal-Bench 3.0, more effort buys verification and edge-case testing, not better approaches
- [[lessons-from-building-claude-code-how-we-use-skills-shihipar-2026]] — Anthropic's internal skill practice: categories, gotchas sections, trigger-style descriptions, on-demand hooks
- [[lessons-from-building-claude-code-prompt-caching-is-everything-shihipar-2026]] — How Claude Code is built around prefix caching: static-first layout, no mid-session swaps, cache-safe compaction
- [[how-we-made-claude-ai-3x-faster-wang-2026]] — Anthropic case study: a Claude-driven two-week sprint made claude.ai ~3x faster using measurable proxies
- [[automating-eval-design-and-hillclimbing-martin-2026]] — Lance Martin on eval design and overfit-safe hill-climbing with the claude-api skill
- [[seeing-like-an-agent-shihipar-2026]] — Claude Code tool-design lessons: structured tools where the harness parses output, grep over RAG, fewer tools
- [[unreasonable-effectiveness-of-html-shihipar-2026]] — Shihipar's opinion piece: HTML rather than Markdown for agent output meant for humans
- [[claude-code-mods]] — Claude Code mods: in-process, stateful hook modules shipped as plugins (Oct 2026)
- [[claude-dev]] — claude.dev, Anthropic's developer blog; hub of the posts ingested from it
- [[claude-opus-5-5]] — Claude Opus 5.5 (Sep 2026): cheaper tokens and cache reads, default effort medium
- [[claude-sonnet-5-5]] — Claude Sonnet 5.5 (2026-09-28): 1M context, same prices as Sonnet 5, stricter thinking semantics
