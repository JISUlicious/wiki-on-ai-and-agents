---
title: "Claude Code mods: official documentation and built-in mod sources (Oct 2026)"
type: source
created: 2026-10-08
updated: 2026-10-08
sources:
  - claude-code-mods-docs-overview.md
  - claude-code-mods-docs-create.md
  - claude-code-mods-docs-events.md
  - claude-code-mods-docs-interface.md
  - claude-code-mods-docs-api.md
  - claude-code-mods-docs-reference.md
  - claude-code-mods-docs-admin.md
  - claude-code-mods-docs-test.md
  - claude-code-mods-docs-troubleshoot.md
  - claude-code-mods-repo-readme.md
  - claude-code-changelog-2026-10-06.md
authors:
  - "[[anthropic]]"
introduces:
  - "[[claude-code-mods]]"
year: 2026
venue: "code.claude.com docs; github.com/anthropics/claude-code/mods"
url: https://code.claude.com/docs/en/plugins/mods/overview
tags:
  - 2026
  - docs
status: complete
importance: high
---

# Claude Code mods: official documentation and built-in mod sources

**Sources**:
- Nine pages of the official docs under [code.claude.com/docs/en/plugins/mods/](https://code.claude.com/docs/en/plugins/mods/overview): overview, create, events, interface, api, reference, admin, test, troubleshoot. Fetched as markdown on 2026-10-08.
- The README of the public [`mods/`](https://github.com/anthropics/claude-code/tree/main/mods) folder in anthropics/claude-code, which holds the source of the built-in mods.
- The CHANGELOG for 2.1.287–2.1.291 ([[claude-code-changelog-2026]]).

**Publisher**: [[anthropic]]
**Release**: v2.1.287, published to npm **2026-10-01**. The Desktop app has had it since v2.1.286. The changelog line reads "Added Claude Mods: plugins may now modify deeper behavior."

This complements [[getting-started-with-claude-code-mods-osmani-2026]], a tutorial, with the reference-level and admin material.

## What a mod is

"A mod is a [plugin] that changes how Claude Code looks and behaves. It's made of JavaScript or TypeScript event handlers: Claude Code calls one when an event happens… and the handler can watch the event, change it, or take it over."

The docs disambiguate the names. Both kinds are "hooks", so on these pages:
- **hook** means a mod's in-process handler;
- **settings hook** means the older kind, a shell command, HTTP request or prompt configured in a settings file.

**Files:**
- `.claude-plugin/plugin.json`;
- `hooks/hooks.json` with `"modules": ["./register.js"]`. The same file can also hold settings hooks.
- an ES module exporting `register(on, options)`, where `options` holds the plugin's `userConfig` values;
- optionally `types/index.d.ts` and `*.test.ts` tests.

**The hook signature:** `on(event, matcher?, async ($, e, next) => result)`.
- `$` is the mods API.
- `e` is the event, deeply frozen: pass a modified copy to `next` to change it.
- `next(e)` runs the remaining hooks and then Claude Code's own behaviour, like middleware.

A hook can do three things with an event:
- **observe** it (call `next(e)` and look at the result);
- **rewrite** it (`next({...e, …})`);
- **answer** it (return without calling `next`).

Extras on `next`: `next.signal` (abort), `next.origin` (who fired the event and at which tier), `next.budget` (time left), `next.to(e, tier)` (skip ahead, for organization mods only), and `.catch` handlers with `next.error`.

## What it can hook (reference: about 40 events)

| Group | Events |
|---|---|
| **Tools** | `tool.call` (deny, or answer with a result), `tool.check` (allow / ask / deny after the rules and PreToolUse hooks), `tool.describe` (rewrite a description; put a tool behind or out from behind tool search) |
| **Prompts and what Claude reads** | `prompt.submit` (rewrite, add context, drop), `prompt.compose` / `prompt.section` (**the system prompt, section by section**), `prompt.context`, `prompt.attachment` (Claude Code's own reminders), `prompt.mention` (v2.1.290+), `skill.prompt`, `attribution.text`, `prompt.fill/suggest/edit` |
| **Commands and config** | `command.run`, `command.describe`, `config.set`, `config.describe` |
| **Turns** | `turn.start`, `turn.step` (each model request; can change the **model** or **effort** per request), `turn.complete` |
| **Session** | `session.start/end`, **`session.compact`** (can skip compaction), `session.append` (rewrite stored rows), `session.send/receive` (messages between sessions), `session.measure`, attach/detach |
| **Subagents** | `agent.offer` (withhold a subagent type), `agent.spawn` (pick its model or deny it; covers agent-team teammates) |
| **Interface** | `ui.render` (any render site: spinner, tool rows, the question dialog…), `ui.press/input/select`, `ui.focus/scroll/close`, `ui.message`, `ui.fault` (v2.1.289+) |
| **Other mods** | `plugin.register` (inspect or **refuse** a mod as it loads), `engine.create` (add or withhold API namespaces) |
| **Bridges** | every settings-hook event as `classic.<Event>`, and every mods-API call as an event (e.g. `fs.read`, `model.complete`), so a mod can intercept calls from mods below it |

## The mods API (`$`)

| Namespace | What it reaches |
|---|---|
| `ui` | Panes, the band above the prompt, toasts, notices, `ask` |
| `command`, `tool`, `agent` | Register and call commands, tools and subagent types |
| `model` | `complete`, `fork`, `classify` |
| `prompt` | `submit` (as the mod or `asUser`), `compose` |
| `turn` | `abort` |
| `session` | `messages`, `usage`, `compact`, `send` |
| `config`, `settings`, `env` | Configuration and environment |
| `fs`, `http`, `process`, `mcp` | Files, network, processes, MCP servers |
| `store`, `state`, `clock` | A machine-wide key-value store, reactive state, timers |
| `audio`, `telemetry` | Sound and speech; usage records |

Every reach outside the mod's own code goes through `$`. That is why `claude plugin validate` can list a mod's `hooks:` and `calls:` **without running it**.

## Limits

- A hook's own execution time: **10 s**, or 50 ms for `prompt.edit`.
- A `.catch` handler: 1 s.
- `$.process.run`: 30 s by default, at most 10 min.
- `$.model.complete`: 1024 tokens by default, at most 64K.
- File reads and writes: 4 MiB. `$.store`: 4 MiB in total.
- Redraws are throttled to 10/s, or 30/s in the terminal for visible panes.

## Trust and security

The docs are blunt about what a mod can do:
- "Mods aren't sandboxed." A mod "runs with your permissions": it can read and write any file, read secrets in environment and settings files, see every prompt and tool call, rewrite them, **approve tool calls before you're asked**, and spend your usage.
- Turning on Claude Code's sandboxing isolates Claude's Bash commands, but "a process that a mod starts runs outside it."
- A mod can restyle most of the interface **but not the permission prompt**.
- **Review before installing:** `claude plugin validate ./some-mod` lists the events a mod handles and the calls it makes.

**The run order is a middleware chain, outermost first:**
1. the built-in guard `sec-default`, then the organization's `prependPlugins` mods;
2. user-installed mods;
3. `appendPlugins` mods;
4. other built-in mods.

The order of settings hooks within that chain:
- Managed `PreToolUse` hooks run **before** any mod, and their block is final.
- Other `PreToolUse` hooks run **after** the last mod. A mod that answers `tool.call` itself skips them.
- `tool.check` can approve a call that such a hook blocked.

**Defaults for organizations** (admin page):
- Mods are on.
- `sec-default@builtin` loads ahead of user mods on machines with managed settings, or for Team and Enterprise sign-ins. It protects managed hooks, the system prompt, managed CLAUDE.md, settings and managed MCP tools.
- Deny rules hold unless `allowModsToOverrideDenyRules` is set. Everything else stays allowed. In particular, a user mod can approve an `ask`-rule call, and in auto mode an approved call **skips the classifier**.
- Controls:
  - `allowManagedModsOnly`, `allowManagedHooksOnly`, `disableAllHooks`, `disableSideloadFlags`;
  - plugin-marketplace restrictions (a mod is a plugin);
  - `--safe-mode`;
  - organizations can write **policy mods** that refuse other mods through `plugin.register`.
- Network policy covers `$.http.fetch` but not programs a mod starts.

## Built-in mods

Some of Claude Code's own features are now mods:

| Mod | Role |
|---|---|
| `cc-plugin-agents-md` | Loads AGENTS.md |
| `cc-plugin-diff` | The `/diff` pane |
| `cc-plugin-plugin-authoring` | A skill only: Claude writes mods |
| `cc-plugin-sec-default` | The guard |
| `cc-plugin-telemetry` | Analytics |
| `cc-plugin-you-should-know` | Off by default: "a side agent watches your back… flags things you or Claude might miss" |

Source for diff, agents-md, sec-default and telemetry is public, with tests. Built-in mods ignore `disableAllHooks`, `--bare` and `--safe-mode`.

## Authoring and testing

- **Ask Claude for a mod:** the built-in `plugin-authoring` skill writes it under `~/.claude/dev-mods/<session-id>/`. **Hot reloading** is offered per session, and the mod reloads at the end of each turn that changes it.
- **Hand-written mods:** `claude --plugin-dir <dir>` reloads on save.
- **Tests:** `claude plugin test` runs `*.test.ts` against the engine's own `$`, with a 5 s timeout per test.
- **Sample mods** (anthropics/claude-code-playground): token-weather (a context forecast above the prompt), blast-radius (hold risky shell commands and show what they would change), replay-theater (`/replay` of the last turn's edits).

## Where mods run

- **Hooks run everywhere a plugin loads:** the CLI, Desktop, the VS Code chat panel, `claude -p` and the Agent SDK, Remote Control, and cloud sessions (when the plugin carries over).
- **Drawing appears only** in the terminal and Desktop.

## First week (changelog 2.1.287 → 2.1.291)

About 30 mod-related fixes and additions in four releases. Examples:
- `tool.check` gained a `ceiling` (the approval level an organization requires);
- `turn.step` reports server-side tool uses;
- `claude plugin validate` lists gating hooks that lack a `.catch`;
- security fixes: "a user-installed mod could make an organization's guard skip its check" (now unloaded), and "a user-installed mod could get an organization's plugin unloaded".

The rest are many drawing fixes.

## Why it matters for the wiki

- **The harness is now programmable in-process.** It goes beyond the old hooks that run a shell script per event. A third party can rewrite the **system prompt section by section**, the **reminders**, **per-request model and effort**, **compaction** and **subagent spawning**. This is the most direct realization of "the harness as an editable object" ([[agentic-harness-engineering]], [[harness-engineering-self-improvement-weng-2026]]), here edited by people and by Claude through the `plugin-authoring` skill.
- **It converges with framework middleware.** LangChain 1.0 shipped "middleware… hooks in the agent loop" in 2025-10 ([[langchain-history-2024-2025]]). Claude Code's mods are the same pattern applied to a shipping coding agent.
- **The permission model widens.** Mods can approve tool calls, skipping the auto-mode classifier. That widens the attack surface ([[prompt-injection]], supply chain via plugins), and explains why the guard, `validate`'s static listing and the managed-settings controls arrived together.

## Concepts & entities

[[claude-code-mods]] · [[claude-code]] · [[hooks-in-modern-agents]] · [[agent-skills]] · [[subagents]] · [[context-engineering]] · [[agentic-harness-engineering]] · [[guardrailing]] · [[anthropic]]
