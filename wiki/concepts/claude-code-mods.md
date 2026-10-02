---
title: Claude Code Mods
type: concept
created: 2026-10-02
updated: 2026-10-02
sources:
  - getting-started-with-claude-code-mods-osmani-2026.md
  - claude-code-mods-gallery-2026.md
status: draft
importance: medium
---

# Claude Code Mods

An extension mechanism added to [[claude-code|Claude Code]] in v2.1.287 (October 2026), enabled by default. A **mod** is an in-process JavaScript or TypeScript module, shipped inside a plugin, that hooks into the harness and can observe, rewrite or answer its events and draw interface components. Anthropic's announcement post describes mods as hooks. Some of Claude Code's own features are built as mods: `/diff`, AGENTS.md support and `sec-default`.

## How a mod works

- **Packaging.** A mod is a plugin (it has `.claude-plugin/plugin.json`) whose `hooks/hooks.json` names one module under `"modules"`. It is distributed like any plugin, through a marketplace and `/plugin install`.
- **Registration.** The module exports `register(on, options)` and adds handlers with `on(event, matcher?, ($, e, next) => …)`.
- **Lifecycle.** It is loaded once and stays resident for the session, so unlike a settings hook it keeps state between events.
- **Sandbox.** It has no DOM and no Node; everything outside it goes through the `$` API (ui, session, state, fs, process, http, tool, command, model).
- **Middleware chain.** Handlers run in order: this mod, then other plugins, then Claude Code's default behaviour. A handler can *observe* (`await next(e)` and inspect the result), *rewrite* (`next({...e, …})`) or *answer* (return without calling `next`, e.g. `{deny}`, or serve a command or tool itself).
- **Events.** `tool.call`, `prompt.submit`, `turn.start`/`turn.complete`, `session.start`/end, `command.run`, `session.usage`, and `ui.render`, which fires for every interface component as it is drawn.
- **Limits.** Each hook call has a 10-second limit, and mods run with Claude Code's full access. The post itself warns about that second point.

The tutorial's three examples show the range: a context-window gauge, a hold on risky Bash commands until approved, and a step-through replay of a turn's edits.

## How mods relate to the neighbouring mechanisms

| | What it shapes | How it runs |
|---|---|---|
| Settings hooks | harness events | a shell command per event, JSON over stdin/stdout, stateless |
| **Mods** | harness events and UI | in-process, stateful, can rewrite or answer events and register tools or commands |
| Plugins | — | the packaging and distribution unit; a mod *is* a plugin |
| [[agent-skills\|Skills]] | model behaviour | instructions the model chooses to load |
| Subagents | separate model contexts | mods are not agents; they see subagent turns only via `e.agentId` |

The key distinction is that skills change what the **model** does and only work if the model picks them up, while mods change what the **harness** does and run regardless. The two meet at hooks: skills can register on-demand settings hooks.

> [!note] What this changes for the wiki's hooks taxonomy
> [[hooks-in-modern-agents]] treats Claude Code's hooks as `settings.json` shell commands and plugin hooks as mostly observational. Mods add a third kind: plugin-shipped, in-process middleware that can rewrite and answer events, with its own dotted event names. That page predates mods.

The source itself is slightly inconsistent: the `/mods/` gallery calls mods "small TypeScript functions" while the tutorial says JS or TS, and the gallery's hook lists omit events the tutorial's code uses.

## Related

- [[claude-code]] · [[hooks-in-modern-agents]] · [[agent-skills]] · [[guardrailing]] · [[claude-dev]]

## References

- [[getting-started-with-claude-code-mods-osmani-2026]]
