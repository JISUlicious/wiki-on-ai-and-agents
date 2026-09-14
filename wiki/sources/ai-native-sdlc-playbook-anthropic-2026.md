---
title: "The AI-Native SDLC Playbook"
type: source
created: 2026-09-07
updated: 2026-09-07
sources:
  - https://claude.com/blog/the-ai-native-sdlc-playbook
authors:
  - Anthropic
year: 2026
venue: "Anthropic (claude.com blog)"
introduces:
  - [[ai-native-sdlc]]
status: complete
importance: high
tags:
  - 2026
---

# The AI-Native SDLC Playbook

**Publisher**: [[anthropic|Anthropic]] — [claude.com/blog/the-ai-native-sdlc-playbook](https://claude.com/blog/the-ai-native-sdlc-playbook). An operational playbook, not a forecast; contrast with the [[agentic-coding-trends-report-anthropic-2026|Agentic Coding Trends Report]], which predicts.

## The thesis: bottleneck inversion

The traditional SDLC assumed **writing code was the bottleneck**. Agentic AI removes that constraint, so the bottleneck moves to the stages that still run at human speed — planning, review, testing, deployment. The build stage collapses from weeks to hours while the human-speed stages hold their duration, and the shape of the lifecycle inverts.

This is a sharper claim than "AI makes development faster," and it has a specific consequence: **optimizing the build stage further buys nothing.** The return is in restructuring the stages around it.

## The artifact chain is the design

Each stage commits a versioned artifact that triggers the next. That chain doubles as the audit trail:

```
intent.md  →  spec.md  →  plan.md  →  diff + tests  →  PR review record  →  deployment log  →  incident record
                                                                                                      ↓
                                                                                              back to intent.md
```

Every link carries a commit SHA, timestamp, author and approvals. The loop closes: a breached control band in Stage 6 produces a new `intent.md` and re-enters at Stage 1.

## Six stages, and the plays under each

| Stage | Artifact | Core move |
|---|---|---|
| **1 Plan** | `intent.md` | capture intent in the originator's own words — problem, outcome, affected users, constraints, open questions — reviewed by a product owner, committed to `intent/` |
| **2 Design** | `spec.md` | Claude produces requirements *and* design in one session from `intent.md`, constrained by organizational [[agent-skills\|skills]] for brand, security, compliance, UX |
| **3 Build** | `plan.md`, diff | plan mode before any code; auto mode after approval; `CLAUDE.md`; skills; hooks; parallel sessions and subagents |
| **4 Test** | passing checks | give the session its own feedback loop; continuous evals in CI |
| **5 Deploy** | PR record, deploy log | AI in the PR review loop; hooks as approval gates; CI/CD with tiered autonomy |
| **6 Maintain** | incident record → `intent.md` | control-band monitoring, scheduled scans, on-call via chat |

**Stage 3 (Build)** carries the most machinery. Plan mode iterates a written plan until *"someone unfamiliar could implement from plan alone"* before code is generated; the plan is committed. `CLAUDE.md` holds institutional knowledge, initialized with `/init` and culled. Skills encode policy. Hooks provide deterministic enforcement behind the advisory skills. Parallel sessions run in separate git worktrees with the engineer as the only shared element.

**Stage 4** distinguishes two things the wiki should keep separate: the **feedback loop** runs throughout a task as many times as needed, while a **verifier subagent** is a final check. Its bug-fix protocol is the strongest single practice in the document — write the failing test first, confirm it fails for the right reason, commit it, then fix without touching the test, **with a hook blocking the agent from editing test files during the fix.**

**Stage 5** puts every PR through identical ranked review passes (bugs / security / compliance) defined in a `REVIEW.md`, with an explicit Important-vs-Nit threshold and a nit cap. Separation of duties is preserved: the agent cannot approve its own code, and branch protection requires code-owner approval.

**Stage 6** is where it stops being reactive. A deterministic trigger — a control-band breach computed by script, not by a model — invokes Claude with no person in the path. Response tiers live in a version-controlled `bands.yaml` (1σ log, 2σ diagnose read-only, 3σ propose a PR).

## The governance model is the transferable part

Three enforcement layers, deliberately different in strength:

| Layer | Strength | Owner |
|---|---|---|
| **Skills** | **advisory** — violations "rare but not impossible" | policy owner |
| **Hooks** | **deterministic** — blocks, and the block explains itself | platform engineer |
| **Managed settings** | **non-overridable** — engineers cannot bypass | IT admin |

The playbook is explicit that *"skills are advisory only: enforce compliance with hooks or PR review for policies that must always hold."* That is the correct reading, and this wiki has the measurement behind it — [[harness-if-instruction-following-across-instruction-surfaces-huang-2026|Harness-IF]] finds **skill and tool descriptions rank *last* in precedence** when instruction surfaces conflict, below system prompts, project files and user instructions. A policy that must hold does not belong in a skill.

Two further principles: controls shift **left** (policy applied during generation rather than discovered in review) and **down** (enforcement in hooks rather than in review comments). Autonomy is tiered by environment — dev free, staging middle, production gated on a named release manager. *"The agent may act up to the production gate; it cannot pass it."*

## Every stage names leading and lagging measures

Unusual for a vendor playbook and the most reusable feature. Each play specifies a leading indicator (moves early) and a lagging one (confirms the outcome). Examples:

| Play | Leading | Lagging |
|---|---|---|
| Plan | time from first conversation to committed `intent.md` | survival rate of `intent.md` into Stage 2 |
| Plan mode | share of changes merging on first implementation pass | rework cycles; does the merged diff still match `plan.md` |
| `CLAUDE.md` | how often Claude repeats a mistake the file should catch | time to first merged PR for a new team member |
| Skills | time from policy change to merged skill update | PR findings citing policy — *should fall toward zero* |
| Feedback loop | first-pass CI success rate for agent changes | review time per PR; change failure rate |
| PR review | time to first review; comments resolved without human touching the branch | defects caught before merge vs escaping |
| CI/CD | pipeline failures triaged without paging a human | DORA measures |

> [!note] No numbers anywhere
> The playbook makes **no quantitative claims at all** — no baselines, no percentage improvements, no third-party citations. Expectations are stated qualitatively ("should fall to minutes"). That is a deliberate and defensible choice for a playbook, and it means nothing here can be cited as evidence. Its value is the *structure* and the *measurement scheme*, not any result.

## Where it meets the wiki's measurements

**The `CLAUDE.md` rule is the catastrophic-remembering ratchet, prescribed.**

> [!warning] Tension — "when Claude makes a mistake twice, the correction goes into `CLAUDE.md`"
> That rule is exactly the accumulation mechanism [[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti measured]] across 1,867 repositories: agentic instruction files gain **+226% in instruction count and +140% in total size** over their lifetime, because appending is cheap and deleting requires provenance nobody records. Deletion hazard *falls* with instruction age.
>
> The playbook does supply a bound — *"keep under one page; stale content wastes context"* — which is more than most guidance offers. But it specifies **no deletion mechanism, no review cadence for the file, and no way to tell which instructions are still load-bearing**, which are precisely the gaps that make the ratchet run. Chakrabarti's own fix is **prompt comments carrying outcome-grounded rationale**, which removes 99.3% of excess in a verifiable setting. Adding a "why, and what happened" note beside each correction would make the one-page bound enforceable rather than aspirational. See [[catastrophic-remembering]].

**The PR review loop inherits a known failure mode.** Play 5A routes all PRs through AI review passes so human attention concentrates on whether the change matches the plan. [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]] found that a judge's selective accuracy falls **82.8% → 56.9%** as a trajectory deepens *while its coverage rises* — it commits more readily as it degrades, and emits no signal. The playbook's monthly tuning ritual (rate findings, cap nits, exclude CI-enforced items) is the right shape of countermeasure, but it is calibration by human sampling, and it will not catch silent degradation unless someone tracks accuracy conditioned on an abstention protocol.

**The eval guidance is consistent with the wiki's other Anthropic source.** "Collect 20–50 real tasks, gate configuration changes on pass rate, make every production incident a permanent eval" sits squarely with [[demystifying-evals-for-ai-agents-anthropic-2026]]. Note what counts as a configuration change here: model, prompt, `CLAUDE.md`, skills, *and* hooks — i.e. the whole [[agentic-harness-engineering|harness]], not just the model. That is the right unit, and it matches [[harness-vs-model-attribution|this wiki's attribution finding]] that scaffold-only variation moves outcomes 11–15 points with the model held fixed.

**Skill quality is assumed, not measured.** The playbook's skill measure is *time from policy change to merged skill update* — a latency metric. [[evaluating-skills-not-just-agents-kevin-2026|NVIDIA's ACES]] shows structural scan scores predict live skill lift at **ρ = −0.018**, and [[what-keeps-agent-skills-from-being-reusable-zhang-2026|the 138K-skill audit]] finds 91.8% carry a defect with 67% having routing defects. Shipping a skill fast says nothing about whether it fires or helps; a paired with/without run does.

## Related

- [[ai-native-sdlc]] — the concept page this anchors.
- [[agentic-coding-trends-report-anthropic-2026]] — the forecast companion; this is the operational one.
- [[agent-skills]] · [[skill-md-format]] · [[skill-routing]] · [[guardrailing]] — the policy layers.
- [[agentic-harness-engineering]] · [[code-as-harness]] · [[harness-vs-model-attribution]] — hooks, skills and settings *are* the harness.
- [[llm-as-a-judge]] · [[agent-as-a-judge]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]] — the review loop and its known limits.
- [[demystifying-evals-for-ai-agents-anthropic-2026]] · [[agent-evaluation]] — the eval practice.
- [[catastrophic-remembering]] · [[why-does-claude-md-keep-growing-chakrabarti-2026]] — the `CLAUDE.md` tension.
- [[claude-code]] · [[mcp]] · [[anthropic]]

## References

- [The AI-Native SDLC Playbook](https://claude.com/blog/the-ai-native-sdlc-playbook)
- [[why-does-claude-md-keep-growing-chakrabarti-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]] · [[harness-if-instruction-following-across-instruction-surfaces-huang-2026]] · [[evaluating-skills-not-just-agents-kevin-2026]] · [[demystifying-evals-for-ai-agents-anthropic-2026]]

## Related 2026 sources

- [[building-an-ai-native-sdlc-plugin]] — how to build it: a validated plugin skeleton mapping each play to the weakest primitive that holds it
