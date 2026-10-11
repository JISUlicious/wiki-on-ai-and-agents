---
title: "Getting started with Claude Code mods (Osmani 2026)"
type: source
created: 2026-10-02
updated: 2026-10-08
sources:
  - getting-started-with-claude-code-mods-osmani-2026.md
  - claude-code-mods-gallery-2026.md
authors:
  - Addy Osmani
year: 2026
venue: "claude.dev (Anthropic developer blog)"
publisher: Anthropic
status: complete
importance: high
tags:
  - 2026
---

# Getting started with Claude Code mods

**Author**: Addy Osmani · **Date**: 2026-10-01 · **Publisher**: [[anthropic|Anthropic]] (claude.dev developer blog, "Tutorials")
**URL**: https://claude.dev/blog/getting-started-with-claude-code-mods/ (plus the gallery page https://claude.dev/mods/)

## Summary

A tutorial introducing **mods**, a new [[claude-code|Claude Code]] extension point (Claude Code 2.1.287+, on by default). The post's own definition: mods "are hooks, and they ship inside plugins". A mod is a Claude Code plugin whose `hooks/hooks.json` names exactly one JavaScript/TypeScript module under `modules`. The module exports `register(on, options)` and adds hooks with `on(event, matcher?, ($, e, next) => …)`. The module is loaded once, stays resident for the session, and runs in its own sandbox with no DOM and no Node, so all outside access goes through the `$` API (`ui`, `session`, `state`, `store`, `fs`, `process`, `clock`, `http`, `tool`, `command`, `model`, …). The tutorial builds **Token Weather** (an 80-line context-window gauge drawn above the prompt). It then walks through two larger mods: **Blast Radius**, which holds risky Bash commands and offers Proceed/Cancel, and **Replay Theater**, which steps through a turn's edits one diff at a time.

## Key points

- **Middleware chain.** Hooks chain like middleware: your hook, then other plugins, then Claude Code's default behaviour. A hook can do one of three things:
  - **Observe**: `await next(e)`, then inspect the result.
  - **Rewrite**: call `next({...e, command: safer})`.
  - **Answer**: return a value without calling `next`, e.g. `{deny: "…"}`, or serve a command or tool itself.
- **Events**: `tool.call`, `prompt.submit`, `turn.start`/`turn.complete` (`e.agentId` set for subagent turns), `session.start`/end, `command.run`, `session.usage`, and `ui.render`, which fires for *every* interface component as it is drawn (e.g. `AbovePrompt`, `Pane`).
- **Difference from settings hooks**, in the post's words: a settings hook "runs a shell command for each event and passes JSON over stdin and stdout". A mod instead stays resident. It keeps state, draws live UI, and calls back into Claude Code: opening panes, running processes, registering slash commands (`$.command.register`), or registering **tools the model can call**.
- **Dogfooding.** Some built-in features are implemented as mods, with source in `anthropics/claude-code` under `mods/`. The gallery lists three:
  - `/diff`, a pane of uncommitted changes;
  - `AGENTS.md` support, read as project instructions when no CLAUDE.md exists;
  - `sec-default`, which "keeps an organization's policy out of reach of the mods a person installs" on Team/Enterprise plans.
- **Tooling**:
  - Hot reload for `--plugin-dir` sessions.
  - `$.state` holds host-side values that survive reloads. Values are declared in a `.d.ts` "type contract" (`interface PluginState`), and reads made inside render hooks auto-subscribe to redraws.
  - `claude plugin validate` reports statically what each module hooks, calls, reads and writes.
  - `claude plugin test` runs `*.test.ts` files against the real runtime. Test hooks run *after* the mod and stub Claude Code's answers.
  - Claude Code writes per-build type declarations into `.claude-plugin/types/`.
- **Runtime limits**: a hook gets 10 s of its own time per dispatch, but time spent inside `$` calls doesn't count. `next.signal` aborts when the user presses Esc.
- **Distribution** is the same as for plugins: a marketplace (`.claude-plugin/marketplace.json`, which can be a GitHub repo), `/plugin install`, and the Claude directory. A community example on the gallery is Thariq Shihipar's `next-steps` mod, which suggests up to three follow-up prompts.
- **Security caveat (author's own)**: mods run "with the same access Claude Code has" and are written by third parties. Blast Radius is "a safety net, not a permission system": it only reads command text, so `$(…)`, aliases and scripts can bypass it. Use permission rules for hard blocks.

## Mod vs. adjacent extension points

| Mechanism | What it is | Relation to mods |
|---|---|---|
| Settings hooks ([[hooks-in-modern-agents\|hooks]]) | Shell command per event, JSON over stdin/stdout, stateless | Mods are the in-process, stateful, UI-capable form of hooks |
| Plugin | Packaging/distribution unit (`plugin.json`, marketplace) | A mod *is* a plugin; it is the plugin's code-module behaviour |
| [[agent-skills\|Skill]] | Model-invoked folder of instructions, scripts and resources | Skills shape what the *model* does. Mods shape what the *harness* does, and run without the model choosing them |
| Subagent ([[multi-agent]]) | Separate model context spawned via the Agent/Task tool | Mods can observe subagent turns (`e.agentId`) but are not agents |

## Concepts & entities

- [[claude-code]] (host runtime), [[anthropic]] (publisher), [[thariq-shihipar]] (author of the `next-steps` community mod)
- [[hooks-in-modern-agents]]: mods add an in-process "observe / rewrite / answer" middleware layer on top of Claude Code's shell hooks
- [[guardrailing]]: Blast Radius as a soft guard, explicitly not a permission boundary
- [[agent-skills]], [[multi-agent]], [[context-assembly]] (the Token Weather context gauge)

## What it does not show

- No performance, latency or adoption data. This is a tutorial, and the API "can change between releases".
- The sandbox's isolation guarantees are not specified beyond "no DOM and no Node". It is also unclear how `sec-default` constrains user-installed mods.
  - **Answered 2026-10-08 by the official docs** ([[claude-code-mods-docs-2026]]): "Mods aren't sandboxed". A mod runs with the user's full permissions, and "no DOM and no Node" only means everything goes through the `$` API. `sec-default` protects organization-managed hooks, the system prompt, managed CLAUDE.md, settings and managed MCP tools, and keeps deny rules final. A user mod can still approve `ask`-rule calls, and those skip the auto-mode classifier.
- Minor inconsistency: the gallery calls mods "small TypeScript functions", while the post says JS *or* TS modules. The gallery's hook lists also omit some events the tutorial's code uses (`session.start`, `ui.render`).

## References

- https://claude.dev/blog/getting-started-with-claude-code-mods/ · https://claude.dev/mods/
- _Original sources: `sources/getting-started-with-claude-code-mods-osmani-2026.md`, `sources/claude-code-mods-gallery-2026.md`_
