---
title: "Demystifying Evals for AI Agents"
type: source
created: 2026-07-07
updated: 2026-07-07
sources:
  - demystifying-evals-for-ai-agents-anthropic-2026.html
year: 2026
venue: "Anthropic Engineering blog"
introduces:
  - [[agent-evaluation]]
tags:
  - 2026
status: complete
importance: high
---

# Demystifying Evals for AI Agents

**[[anthropic|Anthropic]] Engineering**, published **2026-01-09** — [blog](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents).

The most widely-shared practitioner guide to [[agent-evaluation]], drawn from Anthropic's internal work and frontier customer deployments. Its lasting contribution is a **precise vocabulary** plus a set of hard-won, sometimes counterintuitive design rules.

## Vocabulary (the part everyone cites)

| Term | Definition |
|---|---|
| **Task** (problem, test case) | a single test with defined inputs and success criteria |
| **Trial** | one attempt at a task; run multiple because outputs vary |
| **Grader** | logic scoring some aspect of performance; a task can have several, each with multiple **assertions** (checks) |
| **Transcript** (trace, trajectory) | the complete record of a trial — outputs, tool calls, reasoning, intermediate results |
| **Outcome** | the **final state in the environment**, not what the agent said |
| **Evaluation harness** | infrastructure running evals end-to-end: provisions tools, runs tasks concurrently, records, grades, aggregates |
| **Agent harness** (scaffold) | the system letting a model act as an agent. *"When we evaluate 'an agent,' we're evaluating the harness and the model working together"* |
| **Evaluation suite** | a collection of tasks measuring related capabilities |

The **outcome vs transcript** distinction is the load-bearing one: a flight-booking agent might say *"Your flight has been booked"* while the outcome is whether a reservation actually exists in the environment's database. This maps directly onto [[verifier|verification by execution]] rather than by assertion.

## Three grader types

| Type | Methods | Strengths | Weaknesses |
|---|---|---|---|
| **Code-based** | string match, fail-to-pass/pass-to-pass tests, static analysis, outcome verification, tool-call verification, transcript analysis | fast, cheap, objective, reproducible, debuggable | brittle to valid variations; lacks nuance |
| **Model-based** | rubric scoring, natural-language assertions, pairwise comparison, reference-based, multi-judge consensus | flexible | needs calibration; can hallucinate |
| **Human** | expert review | ground truth | expensive; use judiciously |

Guidance: **deterministic where possible, LLM where necessary, human for validation.**

## Capability vs regression evals

- **Capability ("quality") evals** — *"What can this agent do well?"* Should **start at a low pass rate**, giving teams a hill to climb.
- **Regression evals** — *"Does it still handle what it used to?"* Should sit near **100%**.
- **Graduation**: once a capability eval saturates, it becomes part of the regression suite. Tasks that measured *"can we do this at all?"* start measuring *"can we still do this reliably?"*

## Non-determinism: pass@k vs pass^k

- **pass@k** — probability of ≥1 success in k attempts. Rises with k. Right metric when one success suffices.
- **pass^k** — probability that **all** k trials succeed. Falls with k. At 75% per-trial and k=3, (0.75)³ ≈ **42%**.

*"At k=1 they're identical. By k=10 they tell opposite stories: pass@k approaches 100% while pass^k falls to 0%."* Choose by product requirement — **pass^k is the honest metric for customer-facing agents** where users expect reliability every time.

## By agent type

- **Coding** — deterministic graders are natural (does it run, do tests pass). SWE-bench Verified went from ~40% to >80% in a year. Also grade the transcript for code quality and tool-use behavior.
- **Conversational** — *"the quality of the interaction itself is part of what you're evaluating."* Usually requires a **second LLM simulating the user** (τ-bench, τ²-bench). Multidimensional: ticket resolved (state check) + under 10 turns (transcript constraint) + appropriate tone (LLM rubric).
- **Research** — no binary signal; combine **groundedness** (claims supported by retrieved sources), **coverage** (key facts a good answer must include), and **source quality** (authoritative, not merely first-retrieved).
- **Computer use** — run in a real/sandboxed environment and check achieved outcome (WebArena-style URL/page-state + backend verification). Notes a real DOM-vs-screenshot tradeoff: DOM is fast but token-hungry, screenshots slower but cheaper — Claude for Chrome has evals checking the agent picks the right one.

## The eight-step roadmap, and its sharper claims

**Collecting tasks**
- **20–50 tasks drawn from real failures is a great start.** Early on, effect sizes are large so small samples suffice. *"Evals get harder to build the longer you wait"* — later you're reverse-engineering criteria from a live system.
- A good task is one where **two domain experts would independently reach the same verdict**. Ambiguity becomes noise in metrics.
- **A 0% pass@100 with a frontier model usually means a broken task, not an incapable agent.**
- **Test both directions.** *"One-sided evals create one-sided optimization"* — evaluating only "did it search when it should" yields an agent that searches for everything. Avoid class-imbalanced evals.

**Harness and graders**
- **Isolate every trial.** Shared state causes correlated failures *and* can inflate scores — internally, Claude gained an unfair advantage on some tasks by reading **git history from previous trials**.
- **Build in partial credit.** An agent that identifies the problem and verifies the customer but fails the refund is meaningfully better than one failing immediately.
- **Calibrate LLM judges against human experts**; give the judge an out (return "Unknown"); prefer **one isolated judge per rubric dimension** over one judge grading everything.
- **Make graders hack-resistant.**

**Maintenance**
- **Read the transcripts.** *"We do not take eval scores at face value until someone digs into the details and reads some transcripts."* Failures should seem fair.
- **Watch for saturation** — an eval at 100% tracks regressions but gives no improvement signal.
- Dedicated eval teams own infrastructure; **domain and product teams contribute the tasks**.

> [!important] "Grade what the agent produced, not the path it took"
> The guide's most contrarian claim, and it cuts against much of the eval-tooling industry's emphasis on trajectory matching:
>
> *"There is a common instinct to check that agents followed very specific steps like a sequence of tool calls in the right order. We've found this approach too rigid and results in overly brittle tests, as agents regularly find valid approaches that eval designers didn't anticipate."*
>
> Illustrated by Opus 4.5 solving a τ²-bench flight-booking task **by discovering a loophole in the policy** — it "failed" the eval as written while actually producing a better outcome for the user.

## The eval-bugs warning

Two concrete cases where the *eval* was wrong, not the model:

- **CORE-Bench**: Opus 4.5 initially scored **42%**. An Anthropic researcher found rigid grading (penalizing "96.12" when expecting "96.124991…"), ambiguous specs, and irreproducible stochastic tasks. After fixes and a less constrained scaffold: **95%**.
- **METR** found misconfigured tasks in its time-horizon benchmark that asked agents to hit a stated threshold while grading required *exceeding* it.

## Where evals fit

Automated evals are one input among **production monitoring, user feedback, A/B testing, manual transcript review, and systematic human evaluation**. They test at scale without a production deployment, but *"can create false confidence if it doesn't match real usage patterns"* and require maintenance to avoid drift.

The appendix surveys frameworks (Harbor for containerized trials, Arize Phoenix/AX for tracing plus online/offline eval) with a pointed caveat: frameworks *"are only as good as the eval tasks you run through them."*

## Concepts & entities

- [[agent-evaluation]] — the concept this anchors.
- [[anthropic]] · [[claude-code]] · [[terminal-bench]] · [[swe-bench]] · [[verifier]]

## References

- [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) (2026-01-09)
- _Original source: `sources/demystifying-evals-for-ai-agents-anthropic-2026.html` (+ extracted `.md`)_
