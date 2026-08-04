---
title: Verifier
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - swe-gym-pan-2024.md
  - verification-horizon-qwen-2026.md
  - probe-process-benchmark-hallucination-detection-2026.md
status: draft
importance: high
---

# Verifier

A **verifier** is any component that judges whether some output, action, or trajectory is *correct* — as distinct from the **generator** that produced it. The generator/verifier split is one of the load-bearing structures in modern LLM systems: it powers test-time selection, reward modeling, agentic self-correction, and safety gating.

The organizing question for any verifier is **what reference it checks against**, because that determines both the method available and the ceiling on what it can catch.

## Verification targets, by reference type

| Target | Question | Reference (ground truth) |
|---|---|---|
| **Factual / groundedness** | Are the output's claims true? | external evidence corpus |
| **Faithfulness / conformance** | Did the action follow the plan or intent? | the plan + user intent, in-context |
| **Plan validity** | Is the plan itself right for the goal? | the true goal + a world model |
| **Task success** | Did it actually work? | execution outcome / the world |
| **Policy compliance** | Was it within bounds? | an explicit policy spec |

The first two are **consistency-against-an-available-reference** problems, and are exactly where a decompose → find-evidence → evaluate → localize pipeline like [[probe-process-benchmark-hallucination-detection-2026|PROBE]] fits — only the reference `R` changes between them.

**Plan validity is a level up**: it asks whether `R` is the *right* `R`. A consistency checker cannot answer that by construction — you would need a ground-truth "correct plan," which is precisely what's missing. It can only be *approached* by importing external signal: human-in-the-loop review, execution feedback, or adversarial critique.

## Mechanisms

Different targets demand different machinery — using one LLM judge for everything is the common design error:

- **Deterministic validation** — schema/format checks, tool-argument validation, cost budgets. Cheap and exact; *validate, don't judge*.
- **Execution / verify-by-running** — unit tests, compilers, interpreters. The strongest signal available, and the reason [[code-as-harness|code]] is such a good agent substrate. Used by [[agentcoder-huang-2023|AgentCoder]] and by [[swe-gym-pan-2024|SWE-Gym]], whose verifier is trained on unit-test pass/fail labels.
- **Learned reward / outcome models** — a model trained on trajectory success labels ([[outcome-based-reward]]), used for **test-time best-of-N** selection: SWE-Gym samples multiple rollouts and picks the verifier-best, pushing SWE-Bench Verified to 32.0%.
- **Process supervision** — grading intermediate reasoning steps rather than final answers.
- **LLM-as-judge / consistency checks** — flexible, but only as good as the reference supplied.
- **Internal-state monitoring** — reading what the model *doesn't say*, e.g. the [[global-workspace-theory|J-lens]] catching evaluation-awareness, fabrication, or hidden goals. A distinct modality: it inspects the generator rather than its output.

## Limits

[[verification-horizon-qwen-2026|The verification horizon]] is the central caution: a verifier is a **proxy** for what you actually want, and the proxy–intent gap can't be perfectly closed. Under optimization pressure the gap becomes the attack surface (Goodhart). Practical consequences:

- Prefer verifiers as an **inference-time gate** over an RL reward signal, where pressure to game them is highest.
- **Localization matters** — a score alone isn't actionable; knowing *which* claim or step failed is what enables targeted retry. Most eval frameworks stop at scoring.
- Co-evolving generator and verifier is one response to staleness — see [[red-queen-godel-machine-iacob-2026]].

> [!note] Draft
> Created during a lint pass — `[[verifier]]` was a recurring link with no page. Consolidates an in-conversation analysis; a fuller **verification-target taxonomy** (with agent-layer breakdown and per-target methods) is still to be filed.

## Related

- [[hallucination-detection]] · [[probe-process-benchmark-hallucination-detection-2026]] — factual verification.
- [[code-as-harness]] · [[agentcoder-huang-2023]] — verify-by-running.
- [[swe-gym-pan-2024]] · [[outcome-based-reward]] — learned verifiers and best-of-N.
- [[verification-horizon-qwen-2026]] — why verification has a ceiling.
- [[dynamic-workflows]] — adversarial critique as a verification stage.

## References

- [[swe-gym-pan-2024]] — outcome-supervised verifier, test-time best-of-N.
- [[verification-horizon-qwen-2026]] — the proxy–intent gap.
- [[probe-process-benchmark-hallucination-detection-2026]] — process-based factual verification.

## In practice

[[agent-evaluation]] is where these targets get operationalized by production tooling. Notable mappings: **state-change verification** (does the calendar event actually exist?) is the industry's name for outcome verification and is the dimension current tools most conspicuously lack; **DeepEval's Plan Quality vs Plan Adherence** is the only shipped instance of the plan-validity vs plan-conformance split above; and [[agent-as-a-judge]] supplies the requirement-level intermediate judgments that flat pass/fail cannot.

## Worked example

- [[in-house-data-agent-eval-test-set]] — a test suite whose grader choices are driven directly by the reference-type axis above (exact reference -> code; retrieved corpus -> consistency judge; no-valid-reference -> abstention check).

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[llm-as-a-verifier-kwok-2026]] — training-free general-purpose verifier reading a *continuous* score off scoring-token logits; +16.6 pts over a discrete judge on the same base model
- [[right-in-the-right-way-damani-2026]] — multiplying verifiable reward by a human-likeness discriminator cuts reward hacking to 1.0% from 96.9–99.3% — a direct attack on the proxy-reward problem
- [[coding-agents-replicate-sciml-papers-hans-2026]] — gates task completion on *workspace evidence* rather than the agent's final message — outcome verification in practice
- [[information-access-llm-monitors-arike-2026]] — extract-then-evaluate monitoring: filtering the trajectory beats dumping it into the monitor's context (+16.8 pp)
- [[not-all-llm-reasoning-visible-in-cot-baherwani-2026]] — **invisible reasoning**: filler tokens lift accuracy up to +30 pp, and Opus 4.5 satisfies a hidden constraint 33.5%→44.5% with no visible trace — behavioural corroboration that output-only verification has a blind spot
- [[gamut-two-level-meta-rubrics-chen-2026]] — factual *completeness* as a verification target, not just precision
- [[arex-recursive-self-improving-research-baai-2026]] — verification gating the transition *between* research rounds (Accept/Refine/Restart), exploiting the discover-vs-verify asymmetry
