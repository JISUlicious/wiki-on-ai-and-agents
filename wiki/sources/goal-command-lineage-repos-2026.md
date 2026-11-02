---
title: "Goal and loop command lineage: repository check"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - ralph-wiggum-software-engineer-huntley-2025.html
year: 2026
url: https://github.com/openai/codex/tree/main/codex-rs/ext/goal
status: complete
importance: medium
tags:
  - 2026
---

# Goal and loop command lineage: repository check

Read on 2026-11-02 at [openai/codex](https://github.com/openai/codex) `a09812981d`, [anomalyco/opencode](https://github.com/anomalyco/opencode) (formerly sst/opencode) `52c7faa073` and [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli) `6814e00d6b`, plus Claude Code's changelog, npm metadata and the default branch of [TianZuo555/pi-extensions](https://github.com/TianZuo555/pi-extensions). Docs and source were read; nothing was run. This extends [[agent-goal-loops-extension-docs-2026]].

## Timeline

| Date | Event | Location |
|---|---|---|
| 2025-07-14 | [[ralph-loop\|Ralph]]: a Bash loop with no built-in exit condition | [Huntley](https://ghuntley.com/ralph/) |
| 2025-11-16 | Anthropic's `ralph-wiggum` plugin: Stop hook, completion-promise string, max iterations (unlimited by default) | anthropics/claude-code `68f90e05dd` |
| 2026-01-06 | Renamed `ralph-loop` "per legal guidance" | anthropics/claude-plugins-official `44328beed4` |
| 2026-03-06 | Claude Code `/loop`, 2.1.71 | changelog; npm |
| 2026-04-13 | Claude Code `/proactive` "is now an alias for `/loop`", 2.1.105 | changelog; npm |
| 2026-04-30 | Codex CLI `/goal`, 0.128.0 | release notes |
| 2026-05-01 | Hermes `/goal` | [[agent-goal-loops-extension-docs-2026]] |
| 2026-05-11 | Claude Code `/goal`, 2.1.139 | changelog; npm |
| 2026-05-18 | Codex's `ext/goal` crate skeleton | `a80f07ec4a` |
| 2026-05-29 | OpenClaw `/goal` | [[agent-goal-loops-extension-docs-2026]] |
| 2026-08-21 | `@tian.zuo/pi-goal` first on npm | npm registry |
| 2026-10-28 | OpenCode scheduled tasks, experimental | `4585c316c3` |
| 2026-10-29 | OpenCode `/goal` | `8de0323975` |

## Implementations

- **Codex CLI:** `codex-rs/ext/goal` is an in-tree extension built on `codex_extension_api`. `templates/goals/continuation.md` tells the working model to treat completion as unproven and audit the evidence. The model then calls `update_goal`, whose status is limited to complete, blocked, or paused at the user's request. "Judge", "evaluator" and "verdict" do not appear in the crate's source, though "verifiers" appears in the template, in another sense. Files outside the crate were not checked.
- **OpenCode:** `packages/core/src/plugin/goal.ts` is an in-tree plugin, described in a comment as a "Stop-hook style loop". `goal/judge.txt` tells a reviewer step not to call tools or continue the work, and says the assistant's claim is not evidence. `MAX_ROUNDS` is 50 and `MAX_STALLS` is 3. The call is `ctx.session.generate({ sessionID, prompt })`, so it is a separate step on the same session. Whether it uses a different model was not established.
- **Pi:** no first-party goal command. The third-party `@tian.zuo/pi-goal` says it follows Codex's goal architecture using Pi's extension primitives. Its source queues continuations from `agent_settled`, and `agent_before_settle` does not appear in its `index.ts` or `src/runtime.ts`. The model marks completion through `update_goal`. It was read, not run.
- **Antigravity CLI:** Google's consumer agent has a documented `/goal` with self-reported completion and a goal stop hook; its source isn't public. See [[antigravity-cli-docs-2026-11]].
- **Gemini CLI:** no file named for a goal or Ralph at this revision, and the requesting session's keyword search of docs and commands also found none. Hooks exist, including `AfterAgent`.

## Three-turn blocked rule

Codex and OpenClaw both tell the model to mark a goal `blocked` only after the same condition repeats for three consecutive goal turns. Both also use the tool names `get_goal`, `create_goal` and `update_goal`.

- The wording first appears in Codex's template file on 2026-06-01 (`c875bc8a33`) and in OpenClaw's `goal.md` on 2026-07-05 (`f7d7148cf0`). Codex may have had it earlier at another path.
- OpenClaw also supports Codex sessions, but no statement of inheritance was found. This is similarity and ordering only.

## Not recorded as verified

Cursor Automations: the requesting session had search snippets only, and they were not checked here.

## Concepts & entities

- [[harness-extension-and-goal-loop-axes]] · [[ralph-loop]] · [[hooks-in-modern-agents]]
- [[antigravity-cli]] · [[codex-cli]] · [[opencode]] · [[pi-mono]] · [[claude-code]] · [[hermes-agent]] · [[openclaw]]

## References

- [Huntley, "Ralph Wiggum as a 'software engineer'"](https://ghuntley.com/ralph/)
- [Claude Code changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [`@tian.zuo/pi-goal`](https://www.npmjs.com/package/@tian.zuo/pi-goal)
