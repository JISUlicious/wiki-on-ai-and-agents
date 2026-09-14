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

The playbook defines six stages joined by an artifact chain, three enforcement layers of unequal strength, and a leading and lagging measure for every play. It ships no implementation. This guide maps each play onto a Claude Code primitive and gives a complete skeleton that a separate build session can turn into a working plugin.

> [!note] Status: a checked skeleton, not a build
> Written against Claude Code **2.1.268** on 2026-09-14, after reading the [Agent Skills specification](https://agentskills.io/specification), Anthropic's [skill-authoring best practices](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices), and the Claude Code docs for skills, plugins, the plugin reference, marketplaces, hooks and plugin evals.
>
> | Check | Result |
> |---|---|
> | `claude plugin validate --strict` on the plugin root | passes |
> | Agent Skills spec validator on the 7 skills | 2 pass; 5 fail by design (see *Two kinds of skill*) |
> | Spec naming rules and reserved words | 7 of 7 comply |
> | Hook scripts, synthetic inputs | 15 of 15 behave as intended |
> | A real session with a model, or `claude plugin eval` | **not run** — that is the build session's job |

## Six design rules, each with its evidence

**1. Human-initiated steps are skills Claude cannot invoke.** Stages 1–3 and 6 begin with a person, so their skills set `disable-model-invocation: true`. The Claude Code docs state the effect exactly: the description is **not in context**, the skill loads only when you type its command, and if Claude tries to call it anyway the call is blocked. That keeps them out of the routing pool entirely, which matters because actual-use precision falls from **29.6% to 3.3%** as a pool grows from 5 to 100 ([[demystifying-agent-skills-jiang-2026|Jiang et al.]]). In that study downstream success stayed roughly flat, so the cost is wasted context and misfires more than failed tasks; see the Contradiction on [[skill-routing]].

**2. Auto-invoked skills only for knowledge Claude should pull mid-task, and few of them.** This skeleton has two. The description is the whole selection surface, and over half of 138,133 public skills have descriptions that cannot drive routing ([[what-keeps-agent-skills-from-being-reusable-zhang-2026|Zhang et al.]]).

**3. Anything that must hold goes in a hook.** The playbook: "skills are advisory only: enforce compliance with hooks or PR review for policies that must always hold." [[harness-if-instruction-following-across-instruction-surfaces-huang-2026|Harness-IF]] found skill descriptions ranked **last** in precedence (4.56) when instruction surfaces conflicted. That pilot used nine older model builds, so treat the ordering as indicative.

**4. Judgement that must not share the author's context goes in an agent.** The verifier and PR reviewer are subagents whose tool lists exclude Write and Edit.

**5. Non-overridable rules are managed settings, and a plugin cannot ship them.** An admin deploys them. A plugin's own `settings.json` supports only the `agent` and `subagentStatusLine` keys. Managed settings can also restrict which marketplaces engineers may use.

**6. Measure against a no-plugin baseline, never with static scores.** On 62 production skills, static quality scores did not predict live lift at all (Spearman ρ = −0.018, [[evaluating-skills-not-just-agents-kevin-2026|ACES]]). `claude plugin eval` runs every case with and without the plugin by default.

## Two kinds of skill, and the standards each meets

The Agent Skills specification defines six frontmatter fields: `name`, `description`, `license`, `compatibility`, `metadata` and `allowed-tools`. Claude Code accepts about twenty more for skills it loads, plugin skills included. But packaging a skill for claude.ai or the Skills API **fails with a hard error** on any field outside the six.

| | Auto-invoked skills | User-invoked command skills |
|---|---|---|
| In this plugin | `fixing-bugs-test-first`, `applying-org-policy` | `capture-intent`, `write-spec`, `write-plan`, `handle-incident`, `curate-instructions` |
| Frontmatter | the spec's six fields only | adds `disable-model-invocation` and `argument-hint` |
| Portable to other agents and claude.ai | yes; both pass the spec validator | no; declared in a `compatibility` field |
| Files they reference | only inside their own folder | their own `assets/`, plus one shared file at the plugin root |
| Naming | gerund, as the best practices recommend | imperative verb, like the docs' own `/deploy` and `/commit` examples |

The split is forced, not chosen: the spec has no way to say "only a person may invoke this," so any skill that needs that is Claude Code only.

Rules both kinds follow:

- **Names**: lowercase letters, digits and hyphens; no leading, trailing or doubled hyphen; must match the folder; must not contain "claude" or "anthropic". That last rule is why the CLAUDE.md skill is `curate-instructions`.
- **Descriptions** are written in the third person and say what the skill does and when to use it, key use case first. The limit is 1,024 characters, and Claude Code shortens descriptions when the whole listing outgrows its budget.
- **`allowed-tools`** is a space-separated list, as the spec defines it; Claude Code also accepts commas. The grant covers only the turn that invokes the skill and **expires when the user sends their next message**.
- **File references** use relative paths or `${CLAUDE_SKILL_DIR}`, and stay one level deep. Only Claude Code substitutes `${CLAUDE_PLUGIN_ROOT}` in skill text, so only the command skills use it, for the shared interview protocol.
- **Bodies** stay well under the 500-line guideline; detail goes in `references/` or `assets/`.

## Interviews for intent and spec

Stages 1 and 2 interview a person, but in **two separate skills**, not one:

- **The review gate is the point.** A product owner accepts the intent before design starts. One session writing both would turn that gate into a formality, and the playbook's lagging measure — does the intent survive into Stage 2 — would stop meaning anything.
- **Different people answer them.** The originator describes the problem; an engineer or designer settles thresholds and design questions.
- **The chain hook enforces it.** A spec must point to an accepted intent.

| | `capture-intent` | `write-spec` |
|---|---|---|
| Who answers | the person who raised the request | an engineer or designer |
| Asks about | problem, outcome, who is affected, constraints, what they don't know | only the intent's open questions, a measurable threshold for each outcome, and what `applying-org-policy` flags |
| Must not | propose a solution | re-ask anything the intent records |
| "I don't know" | goes under Open questions | stays open, or is resolved with the person |

Both follow one protocol file. The main risk it guards against is a model filling gaps with plausible answers, and one eval case tests exactly that. Because the tool grant expires after the first reply, writing the file at the end of an interview usually raises a permission prompt. That prompt is a useful checkpoint, not a bug.

## Which play goes where

| Playbook element | Component | Primitive | Why |
|---|---|---|---|
| Stage 1: `intent.md` in the originator's words | `/sdlc:capture-intent` | user-invoked skill | a person starts it |
| Stage 2: `spec.md` from an accepted intent | `/sdlc:write-spec` | user-invoked skill | same |
| Stage 2: org rules for security, compliance, brand, UX | `applying-org-policy` | auto-invoked skill | needed mid-task, unprompted |
| Stage 3: plan committed before code | `/sdlc:write-plan` | user-invoked skill | complements built-in plan mode by committing the plan |
| Chain integrity | `chain_check.py` | PostToolUse hook, and the same script in CI | must hold |
| Stage 4: failing test first, test locked | `fixing-bugs-test-first` + `test_lock.py` | auto-invoked skill + PreToolUse hook | the skill teaches; the hook enforces |
| Stage 4: final check against the plan | `verifier` | agent | must not share the author's context |
| Stage 5: ranked review passes | `pr-reviewer` + `REVIEW.md` | agent | same |
| Stage 5: agent cannot approve its own code | `no_self_merge.sh` + branch protection | PreToolUse hook + repository setting | the repository setting is the real control |
| Stage 5: autonomy tiers, production gate | managed settings, CI environments | admin-deployed | engineers must not override it |
| Stage 6: breach → response | `scripts/bands.py` → `/sdlc:handle-incident` | script, then headless Claude | the trigger must be deterministic |
| `CLAUDE.md` upkeep | `claude_md_rationale.py` + `/sdlc:curate-instructions` | PostToolUse hook + user-invoked skill | rationale on entry, outcomes on review |

The last row goes beyond the playbook. Its rule "when Claude makes a mistake twice, the correction goes into `CLAUDE.md`" is the growth mechanism [[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti]] measured at **+226%**. Recording outcome-grounded rationale beside each instruction removed **99.3%** of excess in a verifiable setting, while recording attempts without outcomes was worse than nothing (+70%). See [[catastrophic-remembering]].

## Layout

```
sdlc/
├── .claude-plugin/plugin.json       only the manifest goes in .claude-plugin/
├── README.md
├── shared/interviewing.md           protocol shared by the two interview skills
├── skills/
│   ├── capture-intent/              user-invoked  Stage 1
│   │   ├── SKILL.md
│   │   └── assets/intent-template.md
│   ├── write-spec/                  user-invoked  Stage 2
│   │   ├── SKILL.md
│   │   └── assets/spec-template.md
│   ├── write-plan/                  user-invoked  Stage 3
│   │   ├── SKILL.md
│   │   └── assets/plan-template.md
│   ├── handle-incident/SKILL.md     user-invoked  Stage 6, also run headless
│   ├── curate-instructions/SKILL.md user-invoked  CLAUDE.md upkeep
│   ├── fixing-bugs-test-first/      AUTO, portable  Stage 4
│   │   └── SKILL.md
│   └── applying-org-policy/         AUTO, portable  Stage 2 constraints
│       ├── SKILL.md
│       └── references/{security,compliance,ux}.md
├── agents/{verifier,pr-reviewer}.md
├── hooks/
│   ├── hooks.json
│   ├── test_lock.py  no_self_merge.sh  chain_check.py  claude_md_rationale.py
├── scripts/{check_chain,bands}.py
├── templates/{REVIEW.md,bands.json} copied into the target repository
└── evals/
    ├── bugfix-writes-test-first/
    ├── intent-in-originators-words/
    └── intent-records-unknowns/
```

Plugin skills are namespaced, so `skills/capture-intent` runs as `/sdlc:capture-intent`.

## The files

### Manifest and README

`.claude-plugin/plugin.json`

```json
{
  "name": "sdlc",
  "version": "0.1.0",
  "description": "Implements Anthropic's AI-Native SDLC Playbook in Claude Code: an interviewed intent → spec → plan artifact chain, test-first bug fixing, gated review, and control-band incident response.",
  "author": { "name": "Your Team" },
  "license": "MIT",
  "keywords": ["sdlc", "workflow", "governance"]
}
```

`README.md`

```markdown
# sdlc

A Claude Code plugin implementing Anthropic's AI-Native SDLC Playbook. **Skeleton** — see TODO markers.

| Command | Stage |
|---|---|
| `/sdlc:capture-intent` | 1 — interview the requester, write `intent/…md` |
| `/sdlc:write-spec` | 2 — interview an engineer on open items, write `spec/…md` |
| `/sdlc:write-plan` | 3 — write and commit `plan/…md` before code |
| `/sdlc:handle-incident` | 6 — invoked by `scripts/bands.py` |
| `/sdlc:curate-instructions` | CLAUDE.md upkeep, report-only |

Auto-invoked: `fixing-bugs-test-first`, `applying-org-policy`. Agents: `verifier`, `pr-reviewer`. Hooks enforce the artifact chain, the test lock, no agent self-merge, and a reason on every new CLAUDE.md instruction.

Copy `templates/REVIEW.md` to your repository root, and adapt `templates/bands.json`.
```

### The shared interview protocol

`shared/interviewing.md`

```markdown
# Interview protocol

Used by `capture-intent` and `write-spec`.

1. Ask one question per turn. Stop after eight questions and draft with what you have.
2. Use AskUserQuestion for questions with a small set of answers, such as who is affected or which constraint applies. Offer two to four options; the user can always answer in free text. Ask open questions, such as the problem itself, as plain text.
3. Record answers in the person's own words. Do not paraphrase them into a solution.
4. If the person says they don't know, or is unsure, write the item under **Open questions**. Never fill a gap with a plausible guess.
5. Do not ask for anything the source file already records.
6. Before writing, show the full draft and ask for corrections.

Writing the file usually needs a permission prompt: a skill's pre-approved tools expire once the person sends their next message, so a multi-turn interview ends with a human confirming the write. Treat that prompt as the checkpoint it is.
```

### User-invoked command skills

`skills/capture-intent/SKILL.md`

```markdown
---
name: capture-intent
description: Captures a change request as intent/<date>-<slug>.md by interviewing the person who raised it, in their own words. Stage 1 of the AI-native SDLC.
disable-model-invocation: true
argument-hint: "[one-line summary of the request]"
allowed-tools: Read Glob Write AskUserQuestion
compatibility: Requires Claude Code. Uses disable-model-invocation and plugin path variables, which the Agent Skills spec does not define.
---

# Capture intent (Stage 1)

Request: $ARGUMENTS

Follow the interview protocol in `${CLAUDE_PLUGIN_ROOT}/shared/interviewing.md`.

Ask the person who raised the request for:

- **Problem**: what is going wrong, and for whom
- **Outcome**: what would be true if it were solved
- **Affected users**: who, and how many
- **Constraints**: deadlines, systems that must not change, regulations
- **Open questions**: anything they don't know

Do not propose an implementation. If they describe one, record it under Constraints only if they say it is required.

Write `intent/YYYY-MM-DD-<slug>.md` from `${CLAUDE_SKILL_DIR}/assets/intent-template.md`, with `status: draft` and `owner:` set to the product owner they name. Then stop. A human reviews the file and sets `status: accepted` before `/sdlc:write-spec` can use it.
```

`skills/write-spec/SKILL.md`

```markdown
---
name: write-spec
description: Writes spec/<id>.md from an accepted intent file, interviewing an engineer only about what the intent leaves open. Stage 2 of the AI-native SDLC.
disable-model-invocation: true
argument-hint: "[path to intent/…md]"
allowed-tools: Read Glob Grep Write AskUserQuestion
compatibility: Requires Claude Code. Uses disable-model-invocation and plugin path variables, which the Agent Skills spec does not define.
---

# Write the spec (Stage 2)

Intent file: $ARGUMENTS

1. Read the intent file. If its `status` is not `accepted`, stop and say so.
2. Follow the interview protocol in `${CLAUDE_PLUGIN_ROOT}/shared/interviewing.md`, with the engineer or designer. Ask only about:
   - each item under **Open questions** in the intent
   - a measurable threshold for every outcome, such as a latency, a count, or an error rate
   - requirements the `applying-org-policy` skill flags for this change
3. Write `spec/<intent id>.md` from `${CLAUDE_SKILL_DIR}/assets/spec-template.md`. Set `intent:` to the intent file's path; a hook checks it.
4. Every requirement needs an acceptance criterion a test could check. Anything still unresolved stays under **Open questions** in the spec.
```

`skills/write-plan/SKILL.md`

```markdown
---
name: write-plan
description: Writes and commits plan/<id>.md from an accepted spec before any code is generated. Stage 3 of the AI-native SDLC.
disable-model-invocation: true
argument-hint: "[path to spec/…md]"
allowed-tools: Read Glob Grep Write Bash(git add *) Bash(git commit *)
compatibility: Requires Claude Code. Uses disable-model-invocation and plugin path variables, which the Agent Skills spec does not define.
---

# Write the plan (Stage 3)

Spec file: $ARGUMENTS

1. Read the spec. Write `plan/<id>.md` from `${CLAUDE_SKILL_DIR}/assets/plan-template.md` and set `spec:` to the spec's path.
2. Iterate until someone unfamiliar with the codebase could implement from the plan alone: files to change, order of changes, tests to add, how to roll back.
3. Commit the plan by itself. Implementation happens in a later session, against this committed plan.
```

`skills/handle-incident/SKILL.md`

```markdown
---
name: handle-incident
description: Responds to a control-band breach by diagnosing read-only or proposing a fix PR, then reopens the loop as a new draft intent. Stage 6 of the AI-native SDLC; normally invoked by scripts/bands.py.
disable-model-invocation: true
argument-hint: "[metric] [diagnose|propose-fix]"
allowed-tools: Read Glob Grep Write
compatibility: Requires Claude Code. Uses disable-model-invocation, which the Agent Skills spec does not define.
---

# Handle an incident (Stage 6)

Arguments: $ARGUMENTS

A script detected the breach. Do not second-guess whether it happened.

1. Write `incidents/YYYY-MM-DD-<metric>.md`: the breach, the evidence you found, and your diagnosis.
2. In `propose-fix` mode only, open a pull request with the fix. You may not merge it.
3. Always finish by writing a new `intent/YYYY-MM-DD-<slug>.md` with `status: draft`, describing the problem the incident revealed. That closes the loop to Stage 1.
```

`skills/curate-instructions/SKILL.md`

```markdown
---
name: curate-instructions
description: Reports CLAUDE.md instructions that have no recorded outcome as deletion candidates. Report-only; never deletes.
disable-model-invocation: true
allowed-tools: Read Glob Grep Bash(git log *)
compatibility: Requires Claude Code. Uses disable-model-invocation, which the Agent Skills spec does not define.
---

# Curate CLAUDE.md instructions

Each instruction carries a marker such as `<!-- id:c7 why:"CI failed twice on unformatted code" -->`. Events live in `.claude/ledger.jsonl`, one JSON object per line, with `event` set to `added`, `cited` or `outcome`.

1. List instructions whose id has no `cited` or `outcome` event in the last 60 days.
2. For each, show its `why` and its last event, and note anything in `git log` since then suggesting it still matters.
3. Output a table of deletion candidates. Do not edit CLAUDE.md. A person decides.
```

The three templates are plain frontmatter plus section headings; `intent-template.md` carries `id`, `status: draft` and `owner`, `spec-template.md` carries `intent:` and a requirements table with an acceptance-criterion column, and `plan-template.md` carries `spec:` and `base:`.

### Auto-invoked skills

`skills/fixing-bugs-test-first/SKILL.md`

```markdown
---
name: fixing-bugs-test-first
description: Fixes reported bugs and regressions test-first — writes a failing test, locks it, then fixes the code without editing the test. Use when the user reports a bug, a regression, or wrong behavior to be fixed.
allowed-tools: Read Glob Grep Edit Write Bash
---

# Fixing bugs test-first (Stage 4)

1. Reproduce the bug as a failing test. Run it, and confirm it fails for the reason in the report rather than a setup error.
2. Commit the test on its own.
3. Lock it by adding the test file's path, relative to the repository root, as a line in `.sdlc/test-lock`. A hook blocks edits to locked files.
4. Change the code until the locked test passes. If the test itself looks wrong, stop and tell the user instead of working around the lock.
5. When the full suite passes, remove the line from `.sdlc/test-lock` and commit the fix.
```

`skills/applying-org-policy/SKILL.md`

```markdown
---
name: applying-org-policy
description: Loads the organization's security, compliance, brand and UX rules for a change. Use when writing a spec or designing a change that touches authentication, personal data, payments, public UI, or external APIs.
allowed-tools: Read Glob
---

# Applying organization policy

Read only the file the change needs:

- [references/security.md](references/security.md): authentication, secrets, data handling
- [references/compliance.md](references/compliance.md): retention, audit, regulated data
- [references/ux.md](references/ux.md): brand, accessibility, copy

These rules are advisory. A rule that must always hold belongs in a hook or in PR review, not here.
```

The three `references/` files are placeholders marked TODO.

### Agents

Plugin agents accept `name`, `description`, `model`, `effort`, `maxTurns`, `tools`, `disallowedTools`, `skills`, `memory`, `background` and `isolation`.

`agents/verifier.md`

```markdown
---
name: verifier
description: Performs the final check before a change is proposed. Reads the committed plan and the diff, runs the tests, and reports whether the diff still matches the plan. Use at the end of Stage 4; read-only.
model: inherit
maxTurns: 30
tools: Read, Glob, Grep, Bash
disallowedTools: Write, Edit
---

You are the final verifier. You did not write this change.

1. Read `plan/<id>.md` and the diff against the plan's `base:` commit.
2. Run the test suite. Report pass or fail with the exact command.
3. List every place the diff departs from the plan, and every plan step it does not implement.
4. End with `VERDICT: MATCHES PLAN` or `VERDICT: DIVERGES`, with reasons.
```

`agents/pr-reviewer.md`

```markdown
---
name: pr-reviewer
description: Reviews a pull request in the ranked passes defined in REVIEW.md, separating Important findings from capped Nits. Use in Stage 5; read-only and cannot approve.
model: inherit
maxTurns: 40
tools: Read, Glob, Grep, Bash(git diff *), Bash(gh pr view *), Bash(gh pr diff *)
disallowedTools: Write, Edit
---

Review the pull request using the passes, thresholds and nit cap in `REVIEW.md` at the repository root.

- Run each pass separately and in order. Skip anything CI already enforces.
- Label each finding Important or Nit. Stop listing Nits at the cap.
- You cannot approve or merge. Your output is a comment for a human code owner.
```

### Hooks

The `if` field narrows a handler using permission-rule syntax, so a Bash hook spawns only for matching commands. What exit code 2 does depends on the event:

| Event | Exit 2 |
|---|---|
| `PreToolUse` | blocks the tool call; stderr goes to Claude |
| `PostToolUse` | cannot block — the tool already ran; stderr goes to Claude, which must repair the result |

JSON output with `hookSpecificOutput.permissionDecision: "deny"` is the documented alternative for PreToolUse.

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

A script computes the breach; a model is invoked only afterwards. Two standard deviations runs Claude in plan mode, which can read but not change anything. Three grants a narrow tool list that can open a pull request but not merge one.

`scripts/bands.py`

```python
#!/usr/bin/env python3
"""Stage 6 trigger: a script decides that a metric breached its band; a model is invoked only afterwards.
Run from cron or CI:  python3 bands.py <plugin-dir> [bands.json]
bands.json: {"<metric>": {"value_cmd": "<shell command printing a number>", "mean": <m>, "sd": <s>}}"""
import json, subprocess, sys
plugin = sys.argv[1] if len(sys.argv) > 1 else "./sdlc"
bands = json.load(open(sys.argv[2] if len(sys.argv) > 2 else "bands.json"))
for metric, b in bands.items():
    value = float(subprocess.check_output(b["value_cmd"], shell=True, text=True).strip())
    z = abs(value - b["mean"]) / b["sd"]
    if z < 1:
        continue
    if z < 2:
        print(f"[1 sd] {metric}={value}: logged only")
        continue
    mode = "diagnose" if z < 3 else "propose-fix"
    cmd = ["claude", "-p", "--plugin-dir", plugin, f"/sdlc:handle-incident {metric} {mode}"]
    if mode == "diagnose":
        cmd += ["--permission-mode", "plan"]                      # read-only
    else:
        cmd += ["--allowedTools", "Read", "Glob", "Grep", "Write", "Edit",
                "Bash(git checkout -b *)", "Bash(git commit *)", "Bash(gh pr create *)"]
    subprocess.run(cmd, check=False)
```

### Eval cases

A case is a folder holding `prompt.md` and `graders/`. An eval run cannot answer interview questions, so the interview cases give their answers up front and tell the skill not to ask. Under the default with-without comparison, a `tool_used: Skill` grader is reported as a did-it-fire indicator rather than scored.

`evals/bugfix-writes-test-first/prompt.md`

```markdown
---
max_turns: 30
allowed_tools: [Read, Glob, Grep, Edit, Write, Bash, Skill]
---
Bug report: `slugify("Hello  World")` returns "hello--world" but should return "hello-world". Please fix it.
```

`evals/bugfix-writes-test-first/graders/skill-fired.md`

```markdown
---
type: tool_used
tool: Skill
input_match: '"skill"\s*:\s*"(?:[\w-]+:)?fixing-bugs-test-first"'
---
```

`evals/bugfix-writes-test-first/graders/test-before-fix.md`

```markdown
---
type: llm
---
PASS if a failing test for the double-space case was written, run, and committed before any change to the slugify implementation, and the test file was not edited after that commit. FAIL otherwise.
```

`evals/intent-records-unknowns/prompt.md`

```markdown
---
max_turns: 10
allowed_tools: [Read, Glob, Write, Skill]
---
/sdlc:capture-intent Search results feel wrong lately. I don't know which users are affected or when it started. Don't ask me follow-ups; draft from this.
```

`evals/intent-records-unknowns/graders/unknowns-not-invented.md`

```markdown
---
type: llm
---
PASS if the intent file lists "which users are affected" and "when it started" under Open questions, and nowhere states a specific user group, date, or cause the prompt did not give. FAIL if any of those gaps is filled with a guess.
```

`intent-in-originators-words` follows the same shape, passing only if the draft proposes no implementation.

## Validate and test

```bash
claude plugin validate --strict ./sdlc
```

On a plugin with a manifest, this checks `plugin.json`, `hooks/hooks.json`, and the frontmatter of every skill, agent and command. Planting defects showed what it catches and what it doesn't:

| Defect | Caught? |
|---|---|
| Unparseable skill or agent frontmatter | yes, as an error |
| Missing skill description | yes, as a warning (an error under `--strict`) |
| Invalid `hooks.json` | yes |
| Unknown frontmatter field | **no**, even under `--strict` |
| A skill name breaking the spec's rules, or not matching its folder | **no** |

`validate` checks what Claude Code will load, not the Agent Skills specification. For the portable skills, also run the spec's own validator — [skills-ref](https://github.com/agentskills/agentskills/tree/main/skills-ref), or the `quick_validate.py` shipped with Anthropic's skill-creator plugin. Hook scripts are not exercised by either: pipe a synthetic tool-call event into each script and check its exit code.

```bash
echo '{"tool_input":{"command":"gh pr merge 42"}}' | sh hooks/no_self_merge.sh; echo "exit $?"
```

## Evals

Write evaluations before the skills, as the best practices ask: at least three per skill, each tested on Haiku, Sonnet and Opus. The skeleton ships three cases in total, one per behavior most likely to fail.

```bash
claude plugin eval init
```

`init` drafts cases for the plugin through an interview. To run the suite:

```bash
claude plugin eval ./sdlc --allow-tools Bash Write Edit --runs 3 --max-cost-usd 10 --json results.json
```

`--allow-tools` is required here. Eval sessions strip Bash, Write and Edit unless granted, and the bug-fix case needs all three. Add `--trust-plugin` in CI, and repeat with `--model` once per model. The docs list 2.1.269 as the minimum version, though 2.1.268 already has the command. The number to watch is `meanDelta`, the with-plugin score minus the without-plugin score.

Two further tools:

- **Does it fire?** skill-creator's trigger-eval loop: about twenty realistic queries, half near-misses, with the description tuned against a held-out split. Run it on the two auto-invoked skills only.
- **Is it used?** `/skill-doctor` reports seven-day usage and context cost per skill.

Then track each play's leading and lagging measure from the playbook.

## Rollout

| Phase | Ships | Done when |
|---|---|---|
| 0 | nothing — record baselines for each lagging measure | four weeks of data |
| 1 | `capture-intent`, `write-spec`, `write-plan`, templates, `chain_check.py` in a hook and in CI | intents survive into specs; no chain breaks reach main |
| 2 | `fixing-bugs-test-first`, `test_lock.py`, `verifier`, bug-fix evals | with-plugin beats without-plugin on all three models |
| 3 | `pr-reviewer`, `REVIEW.md`, `no_self_merge.sh`, branch protection | review findings sampled against human labels monthly |
| 4 | `claude_md_rationale.py`, `curate-instructions` | `CLAUDE.md` length stops growing |
| 5 | `bands.py`, `handle-incident`, a schedule | a real breach produced an incident record and a new intent |

Each phase stands alone. Phase 4 is useful even if the rest of the playbook is never adopted.

## Distribution

List the plugin in a `.claude-plugin/marketplace.json` at the root of a repository, then `/plugin marketplace add your-org/claude-plugins` and `/plugin install sdlc@your-marketplace`. For a team, commit `extraKnownMarketplaces` and `enabledPlugins` to the project's `.claude/settings.json`; plugins from external sources still need a manual install.

To publish, submit through the in-app form for the **community** marketplace. Review runs the same `claude plugin validate` plus automated safety screening, and approved plugins are pinned to a commit. The **official** marketplace is curated by Anthropic with no application process.

## Limits, stated plainly

- **Hooks guard tools, not intent.** `test_lock.py` blocks Edit and Write on the locked test, but a shell command could still rewrite it. The real guarantee is a CI check that the test file is unchanged between the test commit and the fix commit.
- **The merge rule belongs to the repository.** The hook is defense in depth; branch protection is the control.
- **The reviewer will degrade silently.** An LLM judge's selective accuracy fell from **82.8% to 56.9%** as trajectories deepened while its coverage rose ([[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]]). Only sampling against human labels shows it.
- **Skill descriptions are an attack surface.** Targeting a description raised self-poisoning by **26.9 percentage points** in [[evomal-self-poisoning-self-evolving-coding-agents-wu-2026|EvoMal]]. Review description changes like code.
- **Unverified here:** whether `claude -p "/sdlc:handle-incident …"` invokes a plugin skill as written, and whether the GitHub Action can load plugins. The docs cover neither.
- **The CLAUDE.md "cited" signal is a weak proxy**, which is why `curate-instructions` reports and never deletes.

## Checklist for the build session

1. Copy the skeleton into its own repository; install it with `claude --plugin-dir ./sdlc` and confirm the hooks registered with `/hooks` in an interactive terminal.
2. Write three evals per skill before changing any skill body; run them on Haiku, Sonnet and Opus.
3. Replace the TODO placeholders in `applying-org-policy/references/`.
4. Wire `scripts/check_chain.py` into CI, and add the check that a locked test is unchanged between the test commit and the fix commit.
5. Copy `templates/REVIEW.md` to the target repository root; turn on branch protection with code-owner approval.
6. Replace `templates/bands.json` with real metrics; verify headless invocation of `handle-incident` before scheduling it.
7. Run skill-creator's trigger-eval loop on the two auto-invoked skills.
8. Re-run `claude plugin validate --strict`, the spec validator on the portable skills, and the hook tests after every change.

## What not to build

Anthropic's official plugins already cover these: `hookify` for authoring hook rules, `skill-creator` for skill evals and description tuning, `claude-md-management` for auditing `CLAUDE.md`, the built-in `/code-review`, and `security-guidance` or `claude-security` for security review.

## Related

- [[ai-native-sdlc]] · [[ai-native-sdlc-playbook-anthropic-2026]] · [[agentic-coding-trends-report-anthropic-2026]]
- [[agent-skills]] · [[skill-md-format]] · [[skill-routing]] · [[skill-library]] · [[guardrailing]]
- [[agentic-harness-engineering]] · [[code-as-harness]] · [[harness-vs-model-attribution]] · [[fundamental-components-of-harness]]
- [[catastrophic-remembering]] · [[llm-as-a-judge]] · [[agent-evaluation]] · [[demystifying-evals-for-ai-agents-anthropic-2026]]
- [[claude-code]] · [[claude-code-dynamic-workflows-harness-2026]] · [[multi-agent]]
- [[ai-enabled-cyber-operations]] — the same skill, memory and hook patterns in adversarial use.

## References

- [[ai-native-sdlc-playbook-anthropic-2026]] · [[demystifying-agent-skills-jiang-2026]] · [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] · [[harness-if-instruction-following-across-instruction-surfaces-huang-2026]] · [[evaluating-skills-not-just-agents-kevin-2026]] · [[why-does-claude-md-keep-growing-chakrabarti-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]] · [[evomal-self-poisoning-self-evolving-coding-agents-wu-2026]]
- [Agent Skills specification](https://agentskills.io/specification) · [Skill authoring best practices](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices)
- Claude Code docs (code.claude.com/docs/en): skills, plugins, plugins-reference, plugin-marketplaces, hooks, plugin-evals
