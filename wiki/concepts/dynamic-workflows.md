---
title: Dynamic Workflows (Claude Code)
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - claude-code-dynamic-workflows-2026.md
  - claude-code-dynamic-workflows-harness-2026.md
status: complete
importance: high
builds-on:
  - [[code-as-harness]]
  - [[multi-agent]]
tags:
  - 2026
---

# Dynamic Workflows (Claude Code)

**Dynamic workflows** are [[claude-code|Claude Code]]'s mechanism for letting [[claude|Claude]] **write and orchestrate its own multi-agent harness on the fly**. Given a task description, Claude authors a **JavaScript orchestration script** that spawns and coordinates tens to hundreds of parallel [[multi-agent|subagents]] in a single session; a runtime executes it in the background while the session stays responsive, and results are verified before they reach the user. Introduced by [[anthropic|Anthropic]] in a [[claude-code-dynamic-workflows-2026|May 28 2026 announcement]] and detailed in a [[claude-code-dynamic-workflows-harness-2026|June 2 2026 engineering post]] (Thariq Shihipar & Sid Bidasaria), and enabled by [[claude-opus-4-8|Claude Opus 4.8]]. Available in research preview across the Claude Code CLI, Desktop, and VS Code extension (Max / Team / Enterprise) and via the Claude API, Amazon Bedrock, Vertex AI, and Microsoft Foundry.

This is the **self-authored** point on the [[code-as-harness]] axis: rather than choosing a fixed harness, Claude writes a custom harness — in code — tailored to the task at hand.

## Why: failure modes of single-context execution

The default Claude Code harness must **plan and execute within one context window**. The June post argues this breaks down for long-running, massively parallel, highly structured, or adversarial tasks, naming three failure modes that motivate splitting work across separate subagent contexts:

- **Agentic laziness** — Claude stops before finishing a complex multi-part task and declares it done after partial progress (e.g. addressing 35 of 50 items in a security review).
- **Self-preferential bias** — Claude tends to prefer its own outputs, especially when asked to verify findings against a rubric. (Connects to the LLM-as-judge literature and to [[agentic-misalignment]] concerns about self-assessment.)
- **Goal drift** — gradual loss of fidelity to the original objective across many turns; *"each summarization step is lossy, and details like edge-case requirements or 'don't do X' constraints can get lost."*

Each subagent gets its own clean context window and a focused, isolated goal — so verification is done by an *independent* agent rather than the one that produced the work.

## How it works

A workflow is a JavaScript file with special functions for spawning and coordinating subagents, plus ordinary JS (`JSON`, `Math`, `Array`) for data processing. The runtime lets Claude:

- **Choose each subagent's model** — route simpler steps to Sonnet, complex ones to Opus (intelligence routing).
- **Choose isolation** — run a subagent in its own git **worktree** so parallel file-mutating agents don't collide.
- **Resume** — progress is journaled, so an interrupted session resumes from where it stopped rather than restarting.
- **Return structured output** — fan-out results are merged at a barrier step as structured data.

### Dynamic vs. static workflows

| | Static workflow | Dynamic workflow |
|---|---|---|
| Authored by | Human, ahead of time (via Claude Agent SDK or `claude -p`) | Claude, on the fly per task |
| Generality | Generic — must handle all edge cases | Custom harness specialized to one use case |
| Enabled by | — | [[claude-opus-4-8]] |

## Orchestration patterns

The engineering post catalogs reusable patterns the script encodes:

- **Classify-and-act** — a classifier agent determines task type and routes to different agents/behaviors (and can decide the final output).
- **Fan-out-and-synthesize** — split into small steps, run an agent per step, then a *barrier* synthesize step merges the structured outputs. Good when steps are many or each needs a clean context.
- **Adversarial verification** — for each producing agent, a separate agent verifies its output against a rubric. The core antidote to self-preferential bias.
- **Generate-and-filter** — generate many candidates, filter by rubric/verification, dedupe, return only the highest-quality survivors.
- **Tournament** — agents attempt the same task via different approaches; a judging agent compares pairwise until a winner emerges. Also used to rank/sort large lists qualitatively.
- **Loop-until-done** — for unknown-size work, keep spawning agents until a stop condition (no new findings, no more errors) rather than a fixed pass count.

## Use cases

- **Migrations & refactors** — the canonical demo: Bun was ported from Zig to Rust (Jarred Sumner) — ~750k lines, 99.8% test-suite compatibility, in 11 days; one workflow mapped Rust lifetimes per struct field, another produced behavior-identical `.rs` from `.zig` with parallel agents and dual reviewers per file.
- **Deep research** — the `/deep-research` skill fans out web searches, fetches sources, adversarially verifies claims, and synthesizes a cited report.
- **Deep verification** — extract every factual claim in a report, check each with its own subagent, then a meta-verifier checks the checkers (e.g. verifying a blog draft against the codebase).
- **Codebase-wide audits** — security reviews, auth checks, input validation across a whole codebase with independent verification.
- **Root-cause investigation** — independent hypothesis agents built from *disjoint* evidence (logs / files / data), each facing a panel of verifiers and refuters — countering self-preferential bias in debugging and post-mortems.
- **Triage at scale** — classify, dedupe against tracked issues, act; a **quarantine** pattern bars agents reading untrusted public content from high-privilege actions (a [[guardrailing]] / [[indirect-prompt-injection]] defense). Pair with `/loop` for continuous operation.
- **Memory & rule adherence** — mine recent sessions and code-review comments for recurring corrections, cluster in parallel, adversarially verify, and distill survivors back into `CLAUDE.md` (one verifier agent per rule, plus a skeptic persona to prevent false positives).
- **Evals, sorting, exploration/taste, model routing.**

## Operating it

- **[[ultracode]]** — a `/effort`-menu setting that pairs the `xhigh` [[reasoning-effort|effort]] level with standing permission for Claude to decide *when* to deploy a workflow automatically (otherwise the user asks for one explicitly). Session-scoped; reset with `/effort high`.
- **Combine with `/loop` and `/goal`** — `/loop` for repeatable runs at intervals (triage, research, verification); `/goal` for hard completion requirements.
- **Token budgets** — workflows consume substantially more tokens than normal sessions; prompt "use 10k tokens" to cap, and start with scoped tasks. The posts are explicit that most ordinary coding tasks *don't* need a panel of five reviewers.
- **Saving & sharing** — press "s" in the workflow menu to save to `~/.claude/workflows`, or distribute as an [[agent-skills|Agent Skill]] by putting the `.js` file in the skill folder and referencing it from `SKILL.md` (optionally as a template rather than a verbatim script).

## Relation to other concepts

- [[code-as-harness]] — dynamic workflows are the **self-authored harness** endpoint: the harness is code Claude writes per task. Compare [[agentic-harness-engineering]] (harness *mutated* by telemetry) — dynamic workflows generate a fresh harness instead.
- [[multi-agent]] — the execution substrate; workflows are deterministic JS orchestration over LLM subagents, the inverse of model-driven multi-agent chatter.
- [[agent-three-layer-model]] — a workflow is an L2 control-flow artifact that itself spawns L2 agents.
- [[claude-code]] — the host runtime; subagents reuse Claude Code's `Task`-tool subagent machinery and worktree isolation.
- [[self-improving-agent]] — saving/distilling successful workflows (and the CLAUDE.md-mining pattern) is a lightweight self-improvement loop.

## References

- [[claude-code-dynamic-workflows-2026]] — the announcement (May 28 2026).
- [[claude-code-dynamic-workflows-harness-2026]] — the engineering deep-dive (June 2 2026): failure modes, patterns, use cases.
- [[code-as-harness]] — the framing this extends.
- [[multi-agent]] — the orchestration substrate.
- Announcement: https://claude.com/blog/introducing-dynamic-workflows-in-claude-code
- Engineering: https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code
- Docs: https://code.claude.com/docs/en/workflows
