---
title: "Guide: Building a Claude Code Plugin for Anthropic's AI-Native SDLC Playbook"
type: query
created: 2026-09-14
updated: 2026-09-14
sources:
  - ai-native-sdlc-playbook-anthropic-2026.md
  - demystifying-agent-skills-jiang-2026.md
  - what-keeps-agent-skills-from-being-reusable-zhang-2026.md
  - harness-if-instruction-following-across-instruction-surfaces-huang-2026.md
  - evaluating-skills-not-just-agents-kevin-2026.md
  - why-does-claude-md-keep-growing-chakrabarti-2026.md
  - rehearse-confidence-cliff-autoresearch-ji-2026.md
tags:
  - 2026
status: complete
importance: medium
---

# Guide: Building a Claude Code Plugin for Anthropic's AI-Native SDLC Playbook

> **Question**: a guide to build a plugin, including agent skills, that implements [[anthropic|Anthropic]]'s [[ai-native-sdlc-playbook-anthropic-2026|AI-Native SDLC Playbook]].

The playbook defines six stages joined by an artifact chain, three enforcement layers of unequal strength, and a leading and a lagging measure for every play. It ships no implementation of its own. This guide maps each play onto a Claude Code primitive, gives a working skeleton, and says how to measure whether it helps.

Everything below was checked on Claude Code **2.1.268** on 2026-09-14. The skeleton passes `claude plugin validate --strict`, and every hook script passed synthetic-input tests (15 of 15). It has **not** been run end to end with a model, and no plugin eval has been run.

## Six design rules, each with its evidence

**1. Human-initiated steps are skills that Claude cannot auto-invoke.** Stages 1–3 and 6 begin with a person. Marking their skills `disable-model-invocation: true` makes them slash commands whose descriptions never enter Claude's context, so they cost no routing budget. This matters because routing degrades with pool size: actual-use precision falls from **29.6% to 3.3%** as a skill pool grows from 5 to 100 ([[demystifying-agent-skills-jiang-2026|Jiang et al.]]). In that study downstream success stayed roughly flat, so the cost is wasted context and misfires more than failed tasks; see the Contradiction on [[skill-routing]]. Plugin `commands/` files are the older format and the docs direct new work to `skills/`.

**2. Auto-invoked skills only for knowledge Claude should pull mid-task, and few of them.** This skeleton has two. Descriptions are the entire selection surface, and over half of 138,133 public skills have descriptions that cannot drive routing ([[what-keeps-agent-skills-from-being-reusable-zhang-2026|Zhang et al.]]).

**3. Anything that must hold goes in a hook.** The playbook says so itself: "skills are advisory only: enforce compliance with hooks or PR review for policies that must always hold." [[harness-if-instruction-following-across-instruction-surfaces-huang-2026|Harness-IF]] measured why: in its conflict pilot, skill descriptions ranked **last** in precedence (4.56), below system prompts, project files and user instructions. That pilot used nine older model builds, so treat the ordering as indicative.

**4. Judgement that must not share the author's context goes in an agent.** The verifier and the PR reviewer run as subagents with read-only tools. That gives separation of duties inside a session.

**5. Non-overridable rules are managed settings, and a plugin cannot ship them.** An admin deploys them. A plugin's own `settings.json` supports only the `agent` and `subagentStatusLine` keys. Managed settings can also restrict which marketplaces engineers may use (`strictKnownMarketplaces`, `blockedMarketplaces`).

**6. Measure with a no-plugin baseline, never with static scores.** On 62 production skills, static quality scores did not predict live lift at all (Spearman ρ = −0.018, [[evaluating-skills-not-just-agents-kevin-2026|ACES]]). `claude plugin eval` runs every case with and without the plugin by default.

## Which play goes where

| Playbook element | Component | Primitive | Why this primitive |
|---|---|---|---|
| Stage 1: `intent.md` in the originator's words | `/sdlc:intent` | user-invoked skill | a person starts it; no routing cost |
| Stage 2: `spec.md` from an accepted intent | `/sdlc:spec` | user-invoked skill | same |
| Stage 2: org rules for security, compliance, brand, UX | `org-policy` | **auto-invoked skill** | Claude needs it mid-task, unprompted |
| Stage 3: plan committed before code | `/sdlc:plan` | user-invoked skill | complements built-in plan mode by committing the plan as an artifact |
| Chain integrity: spec→intent, plan→spec | `chain_check.py` | PostToolUse hook, and the same script in CI | must hold |
| Stage 4: failing test first, test locked during the fix | `test-first-bugfix` + `test_lock.py` | **auto-invoked skill** + PreToolUse hook | the skill teaches the protocol; the hook enforces the lock |
| Stage 4: final check against the plan | `verifier` | agent | must not share the author's context |
| Stage 5: ranked review passes from `REVIEW.md` | `pr-reviewer` | agent | same |
| Stage 5: the agent cannot approve its own code | `no_self_merge.sh` + branch protection | PreToolUse hook + repository setting | must hold; the repository setting is the real control |
| Stage 5: autonomy tiers, production gate | managed settings, CI environments | admin-deployed | must not be overridable by engineers |
| Stage 6: control-band breach → response | `scripts/bands.py` → `/sdlc:incident` | script run by cron or CI, invoking headless Claude | the trigger must be deterministic, not a model |
| `CLAUDE.md` upkeep | `claude_md_rationale.py` + `/sdlc:curate` | PostToolUse hook + user-invoked skill | the ratchet fix: rationale on entry, outcomes on review |

The last row goes beyond the playbook. Its rule "when Claude makes a mistake twice, the correction goes into `CLAUDE.md`" is the growth mechanism [[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti]] measured at **+226%** over instruction-file lifetimes. The measured fix is recording outcome-grounded rationale beside each instruction, which removed **99.3%** of excess in a verifiable setting. Recording attempts without outcomes measured worse than recording nothing (+70%), which is why the curation pass looks for outcomes, not just reasons. See [[catastrophic-remembering]].

## Layout

```
sdlc/
├── .claude-plugin/plugin.json
├── skills/
│   ├── intent/SKILL.md              user-invoked   Stage 1
│   ├── spec/SKILL.md                user-invoked   Stage 2
│   ├── plan/SKILL.md                user-invoked   Stage 3
│   ├── incident/SKILL.md            user-invoked   Stage 6, also called headlessly
│   ├── curate/SKILL.md              user-invoked   CLAUDE.md ledger review
│   ├── test-first-bugfix/SKILL.md   AUTO           Stage 4
│   └── org-policy/                  AUTO           Stage 2 constraints
│       ├── SKILL.md
│       └── references/{security,compliance,ux}.md
├── agents/
│   ├── verifier.md                  Stage 4 final check
│   └── pr-reviewer.md               Stage 5 ranked review
├── hooks/
│   ├── hooks.json
│   ├── test_lock.py
│   ├── no_self_merge.sh
│   ├── chain_check.py
│   └── claude_md_rationale.py
├── scripts/
│   ├── check_chain.py               shared by the hook and CI
│   └── bands.py                     Stage 6 trigger
├── templates/{intent,spec,plan,REVIEW}.md
└── evals/
    ├── bugfix-writes-test-first/
    └── intent-in-originators-words/
```

Plugin skills are namespaced, so `skills/intent` is invoked as `/sdlc:intent`.

## The files

### Manifest

`.claude-plugin/plugin.json`

```json
{
  "name": "sdlc",
  "version": "0.1.0",
  "description": "Implements Anthropic's AI-Native SDLC Playbook: an intent → spec → plan artifact chain, test-first bug fixing, gated review, and control-band incident response.",
  "author": { "name": "Your Team" },
  "license": "MIT",
  "keywords": ["sdlc", "workflow", "governance"]
}
```

### A user-invoked stage skill

`disable-model-invocation: true` is what makes it a slash command only. `allowed-tools` pre-approves just what the stage needs.

`skills/intent/SKILL.md`

```markdown
---
name: intent
description: Capture a change request as intent/<date>-<slug>.md in the originator's own words (Stage 1).
disable-model-invocation: true
argument-hint: "<one-line summary of the request>"
allowed-tools: Read, Write, Glob, AskUserQuestion
---

# Stage 1 — capture intent

Create `intent/YYYY-MM-DD-<slug>.md` from `${CLAUDE_PLUGIN_ROOT}/templates/intent.md`.

1. Interview the originator. Record the **problem**, the **outcome** they want, **affected users**, **constraints**, and **open questions** — in their words, not a solution design.
2. Do not propose an implementation. If the originator describes one, record it under constraints only if they insist it is required.
3. Set `status: draft` and `owner:` to the product owner the originator names.
4. Stop. A human reviews and sets `status: accepted`; only then may `/sdlc:spec` run.
```

The `spec`, `plan`, `incident` and `curate` skills follow the same pattern. `spec` refuses unless the intent is `accepted` and writes an `intent:` field. `plan` writes a `spec:` field and commits the plan alone. `incident` always ends by writing a new draft intent, which closes the loop back to Stage 1.

`skills/incident/SKILL.md`

```markdown
---
name: incident
description: Respond to a control-band breach — diagnose read-only or propose a fix PR — and reopen the loop as a new intent (Stage 6).
disable-model-invocation: true
argument-hint: "<metric> <diagnose|propose-fix>"
allowed-tools: Read, Glob, Grep, Write
---

# Stage 6 — control-band response

Invoked headlessly by `scripts/bands.py` when a metric leaves its band; never by a model's own judgement.

1. Write `incidents/<date>-<metric>.md` with the breach, the evidence, and your diagnosis.
2. In `propose-fix` mode only, open a PR. You may not merge it.
3. Always finish by writing a new `intent/…md` (`status: draft`) describing the problem the incident revealed. That closes the loop back to Stage 1.
```

### An auto-invoked skill

The description names the situation, not the skill's contents, because the description is what routing sees.

`skills/test-first-bugfix/SKILL.md`

```markdown
---
name: test-first-bugfix
description: Use when fixing a reported bug or regression. Enforces write-the-failing-test-first, lock the test, then fix without editing it.
allowed-tools: Read, Edit, Write, Bash, Glob, Grep
---

# Test-first bug fixing (Stage 4)

1. Reproduce the bug as a failing test. Run it and confirm it fails **for the reason in the report**, not for a setup error.
2. Commit the test alone.
3. Lock it: append the test file's path to `.sdlc/test-lock`. A hook now blocks Edit/Write to that path.
4. Fix the code until the locked test passes. If you believe the test itself is wrong, stop and tell the user — do not work around the lock.
5. When the full suite passes, remove the path from `.sdlc/test-lock` and commit the fix.
```

`org-policy` keeps its rules in `references/` files that Claude reads only when needed, so the always-loaded part is one description.

### Agents

`agents/verifier.md`

```markdown
---
name: verifier
description: Final check before a change is proposed — reads the committed plan and the diff, runs the tests, and reports whether the diff still matches the plan. Read-only; dispatched at the end of Stage 4.
model: inherit
tools: Read, Glob, Grep, Bash
---

You are the final verifier. You did not write this change.

1. Read `plan/<id>.md` and `git diff` against the plan's base.
2. Run the test suite. Report pass or fail with the exact command.
3. List every place the diff departs from the plan, and every plan step the diff does not implement.
4. You may not edit files. End with VERDICT: MATCHES PLAN, or VERDICT: DIVERGES, with reasons.
```

The `pr-reviewer` agent gets only `git diff`, `gh pr view` and `gh pr diff` for Bash, reads its passes, thresholds and nit cap from `REVIEW.md`, and cannot approve.

### Hooks

The `if` field narrows a Bash hook to one command pattern. Exit code 2 blocks a PreToolUse call, and its stderr is shown to Claude. For PostToolUse the edit has already happened, so exit 2 feeds the message back and Claude must repair the file. JSON output with `hookSpecificOutput.permissionDecision: "deny"` is the documented alternative for PreToolUse.

`hooks/hooks.json`

```json
{
  "description": "sdlc: artifact-chain integrity, test lock during bug fixes, no agent self-merge, rationale on CLAUDE.md edits",
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          { "type": "command", "command": "python3 \"${CLAUDE_PLUGIN_ROOT}/hooks/test_lock.py\"", "timeout": 10 }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "sh \"${CLAUDE_PLUGIN_ROOT}/hooks/no_self_merge.sh\"", "if": "Bash(gh pr merge:*)", "timeout": 10 },
          { "type": "command", "command": "sh \"${CLAUDE_PLUGIN_ROOT}/hooks/no_self_merge.sh\"", "if": "Bash(gh pr review:*)", "timeout": 10 }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          { "type": "command", "command": "python3 \"${CLAUDE_PLUGIN_ROOT}/hooks/chain_check.py\"", "timeout": 10 },
          { "type": "command", "command": "python3 \"${CLAUDE_PLUGIN_ROOT}/hooks/claude_md_rationale.py\"", "timeout": 10 }
        ]
      }
    ]
  }
}
```

`hooks/test_lock.py`

```python
#!/usr/bin/env python3
"""PreToolUse: block Edit/Write to any test file listed in .sdlc/test-lock."""
import json, os, sys
event = json.load(sys.stdin)
path = (event.get("tool_input") or {}).get("file_path", "")
lock = os.path.join(event.get("cwd", "."), ".sdlc", "test-lock")
if path and os.path.exists(lock):
    locked = {os.path.abspath(os.path.join(event.get("cwd", "."), l.strip()))
              for l in open(lock) if l.strip()}
    if os.path.abspath(path) in locked:
        print(f"{path} is locked: it is the failing test for the current bug fix. "
              "Fix the code, not the test. If the test is wrong, stop and tell the user.", file=sys.stderr)
        sys.exit(2)   # exit 2 blocks the tool call; stderr goes to Claude
sys.exit(0)
```

`hooks/no_self_merge.sh`

```sh
#!/bin/sh
# PreToolUse (Bash, gated by "if"): agents may not merge or approve pull requests.
input=$(cat)
case "$input" in
  *"gh pr merge"*|*"--approve"*)
    echo "Agents may not merge or approve PRs. A code owner must. (Branch protection enforces this too.)" >&2
    exit 2 ;;
esac
exit 0
```

`hooks/chain_check.py`

```python
#!/usr/bin/env python3
"""PostToolUse: after writing spec/ or plan/ files, verify the artifact chain."""
import json, os, subprocess, sys
event = json.load(sys.stdin)
path = (event.get("tool_input") or {}).get("file_path", "")
if not any(f"/{d}/" in path for d in ("spec", "plan")):
    sys.exit(0)
check = os.path.join(os.environ.get("CLAUDE_PLUGIN_ROOT", ""), "scripts", "check_chain.py")
r = subprocess.run([sys.executable, check, path], capture_output=True, text=True)
if r.returncode != 0:
    print(r.stdout + r.stderr, file=sys.stderr)
    sys.exit(2)       # PostToolUse: the write happened; stderr tells Claude to repair it
sys.exit(0)
```

`scripts/check_chain.py`

```python
#!/usr/bin/env python3
"""Verify spec→intent and plan→spec links. Used by the hook and by CI: python3 check_chain.py [files...]"""
import os, re, sys
LINK = {"spec": ("intent", "accepted"), "plan": ("spec", None)}
def front(path):
    t = open(path, encoding="utf-8").read()
    m = re.match(r"---\n(.*?)\n---", t, re.S)
    return dict(re.findall(r"^(\w+):\s*(.+)$", m.group(1), re.M)) if m else {}
bad = []
for f in sys.argv[1:]:
    kind = next((k for k in LINK if f"/{k}/" in f or f.startswith(f"{k}/")), None)
    if not kind or not os.path.exists(f):
        continue
    field, need = LINK[kind]
    target = front(f).get(field, "").strip().strip('"')
    if not target or not os.path.exists(target):
        bad.append(f"{f}: '{field}:' must name an existing file (got {target!r})")
    elif need and front(target).get("status", "").strip() != need:
        bad.append(f"{f}: {target} has status {front(target).get('status')!r}, needs {need!r}")
print("\n".join(bad))
sys.exit(1 if bad else 0)
```

`hooks/claude_md_rationale.py`

```python
#!/usr/bin/env python3
"""PostToolUse: new CLAUDE.md instructions must carry an id and a why."""
import json, re, sys
event = json.load(sys.stdin)
ti = event.get("tool_input") or {}
if not ti.get("file_path", "").endswith("CLAUDE.md"):
    sys.exit(0)
new = ti.get("new_string", ti.get("content", ""))
old = ti.get("old_string", "")
added = [l for l in new.splitlines() if l.strip().startswith("- ") and l not in old]
bare = [l.strip() for l in added if not re.search(r"<!--\s*id:\S+\s+why:", l)]
if bare:
    print("New CLAUDE.md instructions need a marker like "
          "<!-- id:c8 why:\"what went wrong\" --> so they can be pruned later:\n  "
          + "\n  ".join(bare), file=sys.stderr)
    sys.exit(2)
sys.exit(0)
```

### The Stage 6 trigger

A script computes the breach, and a model is only invoked afterwards. Two sigma runs Claude in plan mode, so it can read but not change anything. Three sigma grants a narrow tool list that can open a PR but not merge one.

`scripts/bands.py`

```python
#!/usr/bin/env python3
"""Stage 6 trigger. Deterministic: a script computes the breach, not a model.
Run from cron or CI. Reads bands.json: {"metric": {"value_cmd": "...", "mean": m, "sd": s}}"""
import json, subprocess, sys
PLUGIN = sys.argv[1] if len(sys.argv) > 1 else "./sdlc"
for metric, b in json.load(open("bands.json")).items():
    value = float(subprocess.check_output(b["value_cmd"], shell=True, text=True))
    z = abs(value - b["mean"]) / b["sd"]
    if z < 1:
        continue
    if z < 2:
        print(f"[1σ] {metric}={value} logged"); continue
    mode = "diagnose" if z < 3 else "propose-fix"
    cmd = ["claude", "-p", "--plugin-dir", PLUGIN, f"/sdlc:incident {metric} {mode}"]
    cmd += (["--permission-mode", "plan"] if mode == "diagnose"
            else ["--allowedTools", "Read", "Glob", "Grep", "Write", "Edit",
                  "Bash(git checkout -b:*)", "Bash(git commit:*)", "Bash(gh pr create:*)"])
    subprocess.run(cmd, check=False)
```

### An eval case

A case is a folder holding `prompt.md` plus `graders/`. Under the default with-without ablation, a `tool_used: Skill` grader is reported as a did-the-plugin-fire indicator, not scored. The outcome is judged by the other graders.

`evals/bugfix-writes-test-first/prompt.md`

```markdown
---
max_turns: 20
allowed_tools: [Read, Edit, Write, Bash, Glob, Grep, Skill]
---
Bug report: `slugify("Hello  World")` returns "hello--world" but should return "hello-world". Please fix it.
```

`evals/bugfix-writes-test-first/graders/skill-fired.md`

```markdown
---
type: tool_used
tool: Skill
input_match: '"skill"\s*:\s*"sdlc:test-first-bugfix"'
---
```

`evals/bugfix-writes-test-first/graders/test-before-fix.md`

```markdown
---
type: llm
---
PASS if the transcript shows a failing test for the double-space case being written and run, and committed, before any change to the slugify implementation, and the test file is not edited after that commit. FAIL otherwise.
```

## Build and check

```bash
claude plugin validate --strict ./sdlc           # manifest only
claude plugin validate --strict ./sdlc/skills    # skills
claude plugin validate --strict ./sdlc/agents    # agents
claude --plugin-dir ./sdlc                       # load it; run /hooks to confirm the hooks registered
claude plugin eval ./sdlc --runs 3 --max-cost-usd 5 --json results.json
```

> [!warning] Validating the plugin root checks only the manifest
> On 2.1.268, `claude plugin validate --strict` on the plugin root reported `"contents": []`. Skills and agents were validated only when their directories were passed directly, and nothing validates `hooks.json`. Parse it yourself and test each hook script with synthetic input before trusting it, as was done here.

**Measuring skills.** Use two tools, because they answer different questions:

- **Does it fire?** [[anthropic|Anthropic]]'s skill-creator plugin generates about 20 trigger queries, mixing should-trigger with near-miss should-not-trigger, and optimizes the description against a held-out split. Run it on the two auto-invoked skills only; the others never auto-trigger.
- **Does it help?** `claude plugin eval` compares with-plugin and without-plugin runs on real tasks. Its `meanDelta` is the number that matters.
- **Is it used?** `/skill-doctor` shows seven-day usage and context cost per skill, and flags never-invoked skills for pruning.

Then track each play's leading and lagging measure from the playbook. Examples: time from first conversation to committed `intent.md`, then its survival into Stage 2. First-pass CI success for agent changes, then change failure rate. For `CLAUDE.md`, the file's length over time: if the ledger works, growth should level off.

## Rollout

| Phase | Ships | Done when |
|---|---|---|
| 0 | nothing — record baselines for each lagging measure | four weeks of data exist |
| 1 | `intent`, `spec`, `plan`, templates, `chain_check.py` in a hook and in CI | intents survive into specs, and no chain breaks reach main |
| 2 | `test-first-bugfix`, `test_lock.py`, `verifier`, the bug-fix eval | the eval's with-plugin arm beats without-plugin |
| 3 | `pr-reviewer`, `REVIEW.md`, `no_self_merge.sh`, branch protection | review findings sampled against human labels monthly |
| 4 | `claude_md_rationale.py`, `/sdlc:curate` | `CLAUDE.md` length stops growing |
| 5 | `bands.py`, `/sdlc:incident`, cron or CI schedule | a real breach produced an incident record and a new intent |

Each phase stands alone, and phase 4 is useful even if the rest of the playbook is never adopted.

## Distribution

Put a `.claude-plugin/marketplace.json` at the root of a repository that lists the plugin, then install it with `/plugin marketplace add your-org/claude-plugins` and `/plugin install sdlc@your-marketplace`. To offer it to a whole team, commit `extraKnownMarketplaces` and `enabledPlugins` to the project's `.claude/settings.json`. Per the docs, plugins from external sources still need a manual install.

## Limits, stated plainly

- **Hooks guard tools, not intent.** `test_lock.py` blocks Edit and Write on the locked test, but a shell command could still rewrite the file. The real guarantee is a CI check that the test file is unchanged between the test commit and the fix commit.
- **The merge rule belongs to the repository.** `no_self_merge.sh` is defense in depth; branch protection requiring code-owner approval is the control.
- **The reviewer will degrade silently.** An LLM judge's selective accuracy fell from **82.8% to 56.9%** as trajectories deepened while its coverage rose ([[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]]). Sampling findings against human labels, as phase 3 requires, is the only way to see it.
- **Skill descriptions are an attack surface.** Targeting a description rather than a body raised self-poisoning by **26.9 percentage points** in [[evomal-self-poisoning-self-evolving-coding-agents-wu-2026|EvoMal]]. Review description changes like code.
- **Two things here are unverified.** Whether a headless `claude -p "/sdlc:incident …"` invokes a plugin skill as written, and whether the GitHub Action can load plugins. The docs cover neither; test both before relying on phase 5.
- **The `CLAUDE.md` "cited" signal is a weak proxy.** An instruction can be obeyed forever without being cited. That is why `/sdlc:curate` reports and never deletes.

## What not to build

Anthropic's official plugins already cover these, so reuse them instead: `hookify` for authoring hook rules, `skill-creator` for skill evals and description tuning, `claude-md-management` for auditing `CLAUDE.md`, the built-in `/code-review`, and `security-guidance` or `claude-security` for security review.

## Related

- [[ai-native-sdlc]] · [[ai-native-sdlc-playbook-anthropic-2026]] · [[agentic-coding-trends-report-anthropic-2026]]
- [[agent-skills]] · [[skill-md-format]] · [[skill-routing]] · [[skill-library]] · [[guardrailing]]
- [[agentic-harness-engineering]] · [[code-as-harness]] · [[harness-vs-model-attribution]] · [[fundamental-components-of-harness]]
- [[catastrophic-remembering]] · [[llm-as-a-judge]] · [[agent-evaluation]] · [[demystifying-evals-for-ai-agents-anthropic-2026]]
- [[claude-code]] · [[claude-code-dynamic-workflows-harness-2026]] · [[multi-agent]]
- [[ai-enabled-cyber-operations]] — the same skill, memory and hook patterns in adversarial use.

## References

- [[ai-native-sdlc-playbook-anthropic-2026]] · [[demystifying-agent-skills-jiang-2026]] · [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] · [[harness-if-instruction-following-across-instruction-surfaces-huang-2026]] · [[evaluating-skills-not-just-agents-kevin-2026]] · [[why-does-claude-md-keep-growing-chakrabarti-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]] · [[evomal-self-poisoning-self-evolving-coding-agents-wu-2026]]
- Claude Code docs: plugins, plugin evals, skills, hooks, plugin marketplaces (code.claude.com/docs/en).
