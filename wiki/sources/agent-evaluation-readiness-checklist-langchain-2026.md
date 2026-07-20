---
title: "Agent Evaluation Readiness Checklist"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
year: 2026
venue: "LangChain blog"
authors:
  - Victor Moreira
tags:
  - 2026
status: complete
importance: high
---

# Agent Evaluation Readiness Checklist

**Victor Moreira** (Deployed Engineer, LangChain) — [blog](https://www.langchain.com/blog/agent-evaluation-readiness-checklist), **2026-03-27**.

A 31-item practitioner checklist across six phases. Its framing thesis: *"Start with the simplest eval that gives you signal... Only add complexity when you have evidence that simpler approaches are missing real failures."*

## The organizing model

Observability primitives map onto evaluation levels: **single-step = run**, **full-turn = trace**, **multi-turn = thread**. Start at trace level, then layer in run- and thread-level as needed.

Full-turn evals grade **three dimensions**, and the third is the one most often missed:

1. **Final response**
2. **Trajectory** — *"Did the agent take a reasonable path? Not necessarily the exact path you expected, just a valid one"*
3. **State changes** — *"often overlooked but critical for agents that do things, not just say things... don't just check that it said 'Meeting scheduled!' Verify the calendar event actually exists. If it writes code, run the code. If it updates a database, query the rows."*

## Selected items with substance

**Error analysis is the main event.** *"This is where you should spend 60-80% of your eval effort"* (citing Hamel Husain). Process: gather traces -> open coding (note every issue without pre-categorizing) -> categorize into a failure taxonomy -> iterate until no new categories appear. Failure taxonomy: **prompt problem / tool design problem / model limitation / don't know yet**, each with a different fix.

**Capability vs regression evals** — the same distinction [[demystifying-evals-for-ai-agents-anthropic-2026|Anthropic]] draws. *"Without the split, you'll either stop improving because you're only guarding existing behavior, or you'll ship regressions because you're only chasing new capabilities."*

**Grade the outcome, not the path** — explicitly quoting Anthropic. Example: ask `"Did the meeting get scheduled correctly?"` not `"Did it call check_availability before create_event?"` Plus partial credit.

**But track efficiency separately.** The reconciliation that makes both work: measure ratios of **observed steps / ideal steps**, **observed tool calls / ideal**, **observed latency / ideal** — because *"ideal trajectories measure efficiency, not correctness. You still pass an agent that found a creative route, but you can see if it took longer to get there."* And: *"An agent that's 95% accurate but 10x slower might not be an improvement."*

**Guardrails are not evaluators.** An explicit table: guardrails run *during* execution, before the user sees output, in **milliseconds**, to *block* (PII detection, format validation, safety filters). Evaluators run *after*, asynchronously, in **seconds to minutes**, to *measure*. *"Don't confuse the two."*

**Decompose the grader.** Not one monolithic correctness evaluator — a cited team built **five specialized evaluators** (content accuracy, structure, visual formatting, formula scenarios, text quality), each with dimension-appropriate thresholds.

**Prefer binary pass/fail** over 1-5 scales (*"introduces subjective differences between adjacent scores and requires larger sample sizes"*) — then honestly caveats that recent work suggests short scales may align better for LLM judges specifically.

**N-1 testing for multi-turn.** Take real conversation prefixes from production (first N-1 turns), let the agent generate only the final turn. *"This avoids the compounding error problem of fully synthetic multi-turn simulations"* — an implicit critique of simulator-based approaches.

**Cold start**: *"20-50 hand-reviewed examples you're confident in will outperform hundreds of synthetic examples you haven't verified."*

**Don't run whole external benchmarks.** Cherry-pick and adapt tasks from [[terminal-bench|Terminal-Bench]], BFCL. *"Don't run full benchmarks in aggregate."*

**Tool design over prompt tuning.** *"The goal is to make mistakes structurally impossible, not just unlikely. For example, requiring absolute file paths eliminates an entire class of navigation errors."*

**Distinguish task failures from evaluation failures.** Track run status (`complete`/`error`/`timeout`) explicitly — *"a grader that marks a timeout as 'incorrect reasoning' pollutes your signal."*

**Anti-bloat**: *"Every eval applies pressure on your system over time... More evals does not equal better agents."* Prune evals that stop giving signal.

**Check infrastructure first.** A cited team found **a single extraction bug moved their benchmark from 50% to 73%**. *"Infrastructure issues (timeouts, malformed API responses, stale caches) frequently masquerade as reasoning failures."*

## Three timings, and a CI rule

Offline / Online / **Ad-hoc** — the last being exploratory analysis of ingested traces, *"discovering patterns you didn't anticipate"*, and the post's addition beyond the usual offline/online binary.

CI/CD rule of thumb: *"Use cheap code-based graders in CI for every commit. Reserve expensive LLM-as-judge evaluations for preview/production evaluation."*

## Concepts

- [[agent-evaluation]] · [[demystifying-evals-for-ai-agents-anthropic-2026]] · [[langsmith-evaluation-docs]] · [[verifier]]

## References

- [Agent Evaluation Readiness Checklist](https://www.langchain.com/blog/agent-evaluation-readiness-checklist) (2026-03-27)
