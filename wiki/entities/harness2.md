---
title: Harness² (Harness2)
type: entity
created: 2026-10-21
updated: 2026-10-21
sources:
  - harness2-recursive-agent-harnessing-xu-2026.md
tags:
  - product
status: draft
importance: medium
founders:
  - [[google-research]]
builds-on:
  - [[agentic-harness-engineering]]
  - [[opencode]]
---

# Harness² (Harness2)

**Harness²** is Google Research's open-source framework, Apache-2.0 and written in Python, for **[[recursive-agent-harnessing]]**: improving an agent's harness while it works through a stream of new tasks, without changing model weights and without ground-truth answers. Paper: [[harness2-recursive-agent-harnessing-xu-2026]] ([arXiv:2610.14224](https://arxiv.org/abs/2610.14224)). Code: [github.com/google-research/harness2](https://github.com/google-research/harness2). It is "not an officially supported Google product".

## In one paragraph

For each task, Harness² runs the agent once as a reference. An editor model then writes two edited harnesses: an *invariant* one for general agent weaknesses and a *domain* one for this field. Both are run on the same task and compared with the reference, and a composer keeps only the edits the behavioral evidence supports (**Propose–Probe–Compose**). After the task, a reflector stores the adopted harness and evidence-backed lessons in a library, which guides how the next task's harness gets edited. Each task still starts from the base harness.

## What it edits

Eight file-based components:
- system prompt
- guardrails
- memory
- tool guidance
- subagents
- skills
- scripts
- a middleware plugin

Details: [[harness2-recursive-agent-harnessing-xu-2026]].

## Using it

```bash
uv pip install -e '.[render]'
harness2-demo --output runs/demo --k 2        # no credentials; deterministic editors
harness2-setup lab && harness2 --bench lab --k 1 --mode parallel --max-tasks 1
```

- **Requirements:** Linux and Python 3.12. Model-backed runs also need Node.js 20+ and the Google Cloud CLI; the default model setup uses Vertex AI.
- **Executors:** [[opencode|OpenCode]] by default, or Codex with `--substrate cx`.
- **Budget and mode:** `--k` sets the recursion budget; `--mode` is `parallel` or `sequential`.
- **Benchmarks shipped:** JobBench, LAB (legal) and WorkBuddy-Bench.
- **Model roles:** set by `HARNESS2_SOLVER_MODEL`, `HARNESS2_IMPROVER_MODEL` and `HARNESS2_JUDGE_MODEL`.
- **Safety caveat from the README:** executors have unrestricted network access and Codex runs with its sandbox disabled. Restrict network access before evaluating.

## How it runs OpenCode and Codex

Harness² is **neither a skill nor a plugin** for those tools. It is an external Python orchestrator that launches them as headless command-line runs, one fresh workspace or container per run. Before each run it writes the candidate harness into that workspace as the tool's own config files (`harness2/substrate/`). The editor, composer and selector are also OpenCode sessions (Claude Opus 4.8 by default).

| | OpenCode | Codex |
|---|---|---|
| Version | pinned source checkout (`23fb5e05…`) plus a compatibility patch, run through bun | Codex CLI 0.144.6 |
| Command | `bun run … src/index.ts run "<task>" --dir <ws> --model <m> --format json` | `codex exec "<task>" -C <ws> --ephemeral --json --dangerously-bypass-approvals-and-sandbox --dangerously-bypass-hook-trust` |
| Always-on text (C1–C4) | 4 files merged into the agent prompt (≤ 24,000 chars) | all in `AGENTS.md`, the only always-on file (≤ 8,000 chars; Codex silently truncates at 32 KB) |
| Skills (C6) | `skills/<n>/SKILL.md`, via the `skill` tool; trigger line added to `guardrails.md` | `.agents/skills/<n>/SKILL.md` (`description:` required, or the skill is silently dropped); trigger line added to `AGENTS.md` |
| Subagents (C5) | `sub_agents/<n>/AGENT.md` (`mode: subagent`), via the `task` tool | installed as a Codex `[agents.<n>]` role |
| Scripts (C7) | `scripts/<n>.py`, run via bash | same |
| Middleware (C8) | an **OpenCode plugin** `plugin/<n>.ts`, inside the tool loop | **Codex lifecycle hooks** `.codex/hooks.json` (PreToolUse, PostToolUse, Stop, …); printing `{"decision":"block"}` feeds a reason back to the model |

**OpenCode install modes differ by benchmark:**
- JobBench: appended to `AGENTS.md`, with `.opencode/{skills,agent}`;
- WorkBuddy: rendered into a container prompt template;
- LAB: written into `opencode.json` with a tool gate.

The code warns that changing the mode moves the baseline.

**What's portable:** you can't add Harness² to your own coding agent. Its **output** is portable, though: an adopted harness is plain `AGENTS.md`, skills, scripts, subagents and a plugin or hooks file. The Codex hook stub quotes the Claude Code hook schema.

## Results in brief

- **One step** improved every configuration tested, by up to **+11.5** on WorkBuddy-Bench, **+4.5** on JobBench and **+11.8 pts** on LAB.
- **Recursion budget k = 5** added a further 2.6–6.7.
- **The editor can be the same model** as the executor.
- **Not independently replicated.** One benchmark is judged by the same model that does the editing ([[harness2-recursive-agent-harnessing-xu-2026]]).

## Related

- [[meta-harness-lee-2026]] — the offline alternative
- [[harness-continual-learning-kang-2026]] — a concurrent online method
- [[self-improving-agent]] · [[agentic-harness-engineering]]

## References

- [[harness2-recursive-agent-harnessing-xu-2026]]
