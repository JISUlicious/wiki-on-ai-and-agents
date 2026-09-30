---
title: AI-Native SDLC
type: concept
created: 2026-09-07
updated: 2026-09-15
sources:
  - ai-native-sdlc-playbook-anthropic-2026.md
  - agentic-coding-trends-report-anthropic-2026.md
  - swe-refactor-bench-hong-2026.md
  - when-agents-coordinate-destefanis-2026.md
  - same-task-different-work-weinberger-2026.md
  - why-does-claude-md-keep-growing-chakrabarti-2026.md
status: complete
importance: high
---

# AI-Native SDLC

Restructuring the software development lifecycle around the fact that **writing code is no longer the bottleneck**.

The traditional lifecycle assumed implementation was the expensive step and arranged everything else around protecting it. Agentic coding collapses build time from weeks to hours while planning, review, testing and deployment continue to run at human speed — so the constraint **inverts**. The stages survive; their relative cost does not.

That inversion is the whole idea, and it has a sharp consequence: **optimizing the build stage further returns nothing.** The gain is in restructuring the stages around it.

## The artifact chain

[[ai-native-sdlc-playbook-anthropic-2026|Anthropic's playbook]] gives the concrete form. Each stage commits a versioned artifact that triggers the next, and the chain doubles as the audit trail:

```
intent.md → spec.md → plan.md → diff + tests → PR review record → deployment log → incident record
     ↑                                                                                    │
     └────────────────────  control-band breach reopens as intent  ────────────────────────┘
```

Six stages — Plan, Design, Build, Test, Deploy, Maintain — with the loop closing rather than terminating. Every link carries a commit SHA, timestamp, author and approvals, which is what makes the lifecycle auditable without a separate compliance apparatus.

## Three enforcement layers, deliberately unequal

The most transferable part, and the part most often got wrong:

| Layer | Strength | Use for |
|---|---|---|
| **Skills** | advisory | conventions, house style, things that *should* hold |
| **Hooks** | deterministic — blocks and explains itself | anything that *must* hold |
| **Managed settings** | non-overridable by engineers | regulatory and safety gates |

[[harness-if-instruction-following-across-instruction-surfaces-huang-2026|Harness-IF]] supplies the measurement that justifies the hierarchy: **skill and tool descriptions rank last in precedence** when instruction surfaces conflict, below system prompts, project files and user instructions. A policy encoded only as a skill will lose an argument. Put it in a hook.

Two directions of movement: controls shift **left** (policy applied during generation, not discovered in review) and **down** (enforcement in hooks, not in review comments). Autonomy tiers by environment, with the rule that *the agent may act up to the production gate but cannot pass it*.

## What the evidence supports, and what it doesn't

**The role shift is well evidenced.** Engineers report using AI in roughly **60%** of their work, delegating what is easily verifiable, well-defined or repetitive and keeping anything needing organizational context or taste. Gains appear as **output volume** rather than reduced time-per-task.

**Autonomous long-horizon delivery is where the narrative outruns the measurements.** Anthropic's own reporting is the first counterweight: developers can *"fully delegate"* only **0–20%** of tasks, a figure the [[agentic-coding-trends-report-anthropic-2026|Trends Report]] publishes and names *"the collaboration paradox."* Then:

| Claimed in industry framing | Measured |
|---|---|
| agents build entire applications over days | [[swe-refactor-bench-hong-2026]]: **28 of 520 runs (5.4%)** pass whole-repository migration; 13 of 20 tasks unsolved |
| sustained long-horizon coherence | [[merchantbench-long-term-coherence-shi-2026]]: **27.3%** of human over 365 simulated days |
| orchestrator-coordinated multi-agent teams | [[when-agents-coordinate-destefanis-2026]]: naming a coordinator gives **no communication hub and no reliable success gain**; the one significant contrast fails to reproduce under a sealed rerun |
| AI reviews AI, so oversight scales | [[rehearse-confidence-cliff-autoresearch-ji-2026]]: judge accuracy **82.8% → 56.9%** as a trajectory deepens, *while coverage rises* — degrading silently, committing more |

**So the practical shape is throughput under supervision, not autonomy.** Many parallel supervised streams rather than a few unsupervised ones — which the playbook implicitly concedes when it names the parallelism ceiling as *"how many streams one person can review properly."* That relocates the bottleneck onto **human review capacity**, which is precisely the thing the degrading-judge result says cannot be safely automated away.

## Three things that dominate outcomes and rarely appear in SDLC narratives

**The harness, not the model.** Scaffold-only variation moves results **11–15 points** on [[swe-bench-verified|SWE-bench Verified]] with the model fixed, up to ~48 on some leaderboards ([[harness-vs-model-attribution]]). Cost per success swings **5–30×** across harnesses at equal success rate ([[same-task-different-work-weinberger-2026]]). The playbook gets this right in one place worth copying: it treats a change to the model, prompt, `CLAUDE.md`, skills *or* hooks as a configuration change that must re-run the eval suite.

**Prompt wording, independent of content.** Asking for "multiple approaches" inflates reasoning tokens **2.4–7.4×** across six models **with no success gain**. Bounded-efficiency wording — scope, acceptance criteria, stop condition — is neutral-or-better everywhere.

**Instruction-file entropy.** The new maintenance burden. See below.

> [!warning] The lifecycle accumulates a cost the traditional one did not have
> The standard prescription — *"when the agent makes a mistake twice, the correction goes into `CLAUDE.md`"* — is exactly the mechanism [[why-does-claude-md-keep-growing-chakrabarti-2026|measured]] to grow agentic instruction files **+226% in count and +140% in size** over their lifetime, with deletion hazard *falling* as instructions age. Redundancy makes honest pruning exponential, and **better models ratchet harder**.
>
> A page-length cap helps only if paired with a way to tell which instructions still earn their place. The measured fix is **outcome-grounded rationale** recorded beside each instruction — comments carrying what went wrong and what happened after, which removes 99.3% of excess in a verifiable setting. Note the corollary: attempts recorded *without* outcomes are **worse than no comment at all**. See [[catastrophic-remembering]].

## Reading the vendor literature

> [!note] Provenance
> The two anchor sources here are vendor documents. The [[ai-native-sdlc-playbook-anthropic-2026|playbook]] makes **no quantitative claims at all** — a defensible choice, and it means the structure is the contribution, not any result. The [[agentic-coding-trends-report-anthropic-2026|Trends Report]] does carry numbers but has **no methodology section**, and all seven case studies are customers. Both are valuable as statements of direction and as descriptions of current practice; neither is evidence for its forecasts. Where this wiki holds a controlled measurement on the same question, prefer the measurement.

## Related

- [[ai-native-sdlc-playbook-anthropic-2026]] — the operational anchor: six stages, artifact chain, leading/lagging measures per play.
- [[agentic-coding-trends-report-anthropic-2026]] — the forecast companion, and the source of the 0–20% delegation figure.
- [[agentic-harness-engineering]] · [[code-as-harness]] · [[harness-vs-model-attribution]]
- [[agent-skills]] · [[skill-routing]] · [[guardrailing]] — the policy layers and their real precedence.
- [[catastrophic-remembering]] — the accumulating cost.
- [[llm-as-a-judge]] · [[agent-as-a-judge]] · [[demystifying-evals-for-ai-agents-anthropic-2026]] — oversight and evals.
- [[swe-bench]] · [[swe-refactor-bench-hong-2026]] · [[terminal-bench]] — how the autonomy claim gets measured.
- [[claude-code]] · [[multi-agent]] · [[llm-agent]]

## References

- [[ai-native-sdlc-playbook-anthropic-2026]] · [[agentic-coding-trends-report-anthropic-2026]] · [[swe-refactor-bench-hong-2026]] · [[when-agents-coordinate-destefanis-2026]] · [[same-task-different-work-weinberger-2026]] · [[why-does-claude-md-keep-growing-chakrabarti-2026]] · [[merchantbench-long-term-coherence-shi-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]] · [[harness-if-instruction-following-across-instruction-surfaces-huang-2026]]

## Related 2026 sources

- [[building-an-ai-native-sdlc-plugin]] — a build guide: this playbook implemented as a Claude Code plugin — user-invoked skills for the artifact chain, two auto-invoked skills, hooks for the must-hold rules, reviewer and verifier agents, and plugin evals with a no-plugin baseline
- [[design-docs-are-all-you-need-kushnir-2026]] — the artifact chain taken to its limit: the library's main branch holds almost no code, humans edit only 50 design docs, and coding sub-agents regenerate everything for about $100 a rebuild. The headline correctness result appears only in the abstract
