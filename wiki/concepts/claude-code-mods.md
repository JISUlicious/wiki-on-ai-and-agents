---
title: Claude Code Mods
type: concept
created: 2026-10-02
updated: 2026-10-08
sources:
  - getting-started-with-claude-code-mods-osmani-2026.md
  - claude-code-mods-gallery-2026.md
  - claude-code-mods-docs-overview.md
  - claude-code-mods-docs-reference.md
  - claude-code-mods-docs-admin.md
  - claude-code-mods-docs-events.md
  - claude-code-mods-repo-readme.md
  - claude-code-changelog-2026-10-06.md
status: complete
importance: high
---

# Claude Code Mods

**Mods** let a [[claude-code|Claude Code]] plugin change how the harness *looks and behaves* from inside its own process. They shipped in **v2.1.287 (npm, 2026-10-01)**; the Desktop app has had them since v2.1.286. They are on by default. The changelog line: "Added Claude Mods: plugins may now modify deeper behavior."

A mod is a plugin whose `hooks/hooks.json` names a JavaScript or TypeScript **hooks module**. The module exports `register(on, options)`, and each `on(event, matcher?, async ($, e, next) => …)` adds a handler that Claude Code calls when that event happens.

Primary sources:
- the official docs ([[claude-code-mods-docs-2026]]);
- Anthropic's tutorial ([[getting-started-with-claude-code-mods-osmani-2026]]).

## How it works

- **Loading:** a mod is loaded once and stays resident for the session, so module-level variables persist and hooks share state. A settings hook, by contrast, is a stateless shell command per event.
- **Middleware:** each handler can **observe** (`next(e)`), **rewrite** (`next({...e, …})`) or **answer** (return without calling `next`: deny a tool call, return a tool result, drop a prompt, serve a command). Hooks on one event form a chain, outermost first.
- **The `$` API is the only way out:**
  - interface: `ui`;
  - extension points: `command`, `tool`, `agent`;
  - models and turns: `model`, `prompt`, `turn`, `session`;
  - system access: `fs`, `http`, `process`, `mcp`;
  - state: `store`, `state`, `clock`;
  - also `config`, `settings`, `env`, `audio`, `telemetry`.

  Because every outside effect is an API call, `claude plugin validate` can list a mod's events and calls **without running it**.
- **About 40 events:** tools, prompts and the **system prompt by section**, reminders, commands, turns (per-request **model and effort**), session (including **compaction** and messages between sessions), subagent spawning, every interface render site, other mods loading (`plugin.register` can refuse a mod), and bridges to every settings-hook event and API call.
- **Limits:** 10 s per hook (50 ms for `prompt.edit`), 1 s for `.catch` handlers, 4 MiB file I/O. A failing hook is skipped unless it has a `.catch`, which is how a guard is made to fail closed.

> [!warning] Contradiction (resolved by the official docs)
> **Sandboxing.** This page previously said, after the tutorial, that a mod runs in a "sandbox" with no DOM and no Node. The official docs say **"Mods aren't sandboxed"**: a mod "runs with your permissions" and can read and write files, read secrets, start processes, reach the network, approve tool calls, and spend usage. Turning on Claude Code's sandbox does not cover processes a mod starts.
>
> The accurate reading: a mod has no direct Node or DOM APIs and reaches everything through `$`, but `$` gives it full user-level access.
>
> **Events.** The earlier page listed a `session.usage` event. The reference has a **`session.measure`** event, plus a `$.session.usage()` **method**.

## Security model

- **Trust:** install only from trusted authors and marketplaces, and run `claude plugin validate` first. A mod cannot change the **permission prompt**, but it can approve or deny a call *before* the prompt appears. In auto mode, a call a mod approves **skips the classifier**.
- **The run order:**
  1. built-in guard `sec-default` → organization `prependPlugins`;
  2. user mods;
  3. `appendPlugins`;
  4. other built-ins.
  Managed `PreToolUse` hooks run before every mod, and their blocks are final.
- **Built-in guard `sec-default`:** loads on managed machines and for Team/Enterprise sign-ins. It protects managed hooks, the system prompt, managed CLAUDE.md, settings and managed MCP tools. Deny rules hold unless `allowModsToOverrideDenyRules` is set.
- **Organization controls:**
  - `allowManagedModsOnly`, `allowManagedHooksOnly`, `disableAllHooks`, `disableSideloadFlags`;
  - marketplace restrictions;
  - custom **policy mods**.
- **First-week fixes included guard bypasses:** "a user-installed mod could make an organization's guard skip its check", and could get an organization's plugin unloaded (changelog 2.1.288–2.1.290).

## Built-in mods

Some of Claude Code's own features are now mods. Source for the first four is public in `anthropics/claude-code/mods`.

| Mod | Role |
|---|---|
| `agents-md` | Loads AGENTS.md |
| `diff` | The `/diff` pane |
| `sec-default` | The guard |
| `telemetry` | Analytics |
| `plugin-authoring` | A skill that lets **Claude write mods**, with per-session hot reload from `~/.claude/dev-mods/<session-id>/` |
| `you-should-know` | Off by default: a side agent that "watches your back" during long tasks and shows notes above the prompt |

## How mods relate to the neighbouring mechanisms

The official comparison table:

| | Mod | Settings hook | Skill | MCP server |
|---|---|---|---|---|
| Runs as | Functions inside Claude Code's process | Shell command, HTTP request or prompt per event | `SKILL.md` instructions Claude reads | External process giving Claude tools |
| Can change | Tool calls, prompts, commands, turns, interface | Go/no-go, tool arguments and results, added context | What Claude knows and does | Which tools Claude has |
| Draws UI | Yes | No | No | No |
| Pick it when | You want a pane, a band, a custom command, or to rewrite events | You want to block, allow or log with an existing script | You keep pasting the same instructions | Claude needs an external system |

**The key distinction:** [[agent-skills|skills]] change what the **model** does, and only if the model loads them. Mods change what the **harness** does, regardless of the model. One plugin can ship a mod, a skill and an MCP server together.

[[hooks-in-modern-agents]] now has three kinds of Claude Code hook:
- settings hooks (2025-06-30);
- plugin-shipped settings hooks;
- mods (2026-10-01): in-process middleware that can rewrite and answer events.

## Why it matters

- **The harness becomes programmable.** Mods expose the system prompt, reminders, per-request model and effort, compaction and subagent spawning as hook points. That makes the harness an editable object at run time ([[agentic-harness-engineering]]), and Claude can edit it through the authoring skill. It is a concrete product form of [[context-engineering]] hooks.
- **It converges with agent-framework middleware.** Compare LangChain 1.0's middleware ([[langchain-history-2024-2025]], 2025-10-22).
- **It widens the trust surface.** In-process code with the user's full permissions and the ability to approve tool calls is a supply-chain and [[prompt-injection]] concern. Hence the guard, static validation and managed controls.

## Related

[[claude-code]] · [[hooks-in-modern-agents]] · [[agent-skills]] · [[subagents]] · [[guardrailing]] · [[claude-dev]] · [[agentic-harness-engineering]]

## References

- [[claude-code-mods-docs-2026]]
- [[getting-started-with-claude-code-mods-osmani-2026]]
- [[claude-code-changelog-2026]]
