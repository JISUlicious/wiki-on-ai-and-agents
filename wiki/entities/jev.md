---
title: Jev
type: entity
created: 2026-09-21
updated: 2026-10-11
sources:
  - introducing-system-one-models-and-jev-typesafe-2026.md
  - independent-jev-calibration-audits-2026.md
  - system-one-ecosystem-sweep-2026-09-28.md
  - jev-research-papers-september-2026.md
status: complete
importance: medium
tags:
  - model
---

# Jev

[[typesafe-ai|TypeSafe AI]]'s first [[system-one-model|System One model]], announced 2026-09-15 and named after the economist William Stanley Jevons. It answers questions whose answer space is declared in advance and returns typed values with probabilities. It generates no text.

- **Interface**: program state plus schema-defined questions in one request; all questions answered in a single parallel pass. `POST /v1/systemone`, currently `jev-1.13.0`.
- **Limits**: 64k tokens per request, 32k for the state plus the longest question; text input only.
- **Training**: RLCD, [[reinforcement-learning|Reinforcement Learning]] for Calibrated Decisions. No details published.
- **Reported performance**: 70–500 ms per call and 67.8% accuracy on a four-workflow benchmark against 67.9% for GPT-5.6 Terra and 74.1% for Sol. All vendor-reported; see the source page for the caveats, which are substantial.

**Open reimplementations**: within four days of the announcement, [[nimble|Bespoke-Nimble-9B]] (a fine-tune) and [[openjev|OpenJev]] (reading [[diffusiongemma|DiffusionGemma]] untrained) reproduced the interface, the latter wire-compatible enough that TypeSafe's own SDK runs against it unchanged. Measured agreement landed within ~3 points of Jev on Jev's own public cases. See [[logit-scoring]].

**What it cannot do**: free text, code, open-ended reasoning, or any answer outside the declared space. It returns no rationale, so a decision it makes cannot be explained to an auditor from its output alone.

## Calibration, as measured

Seven independent audits between 2026-09-17 and 2026-09-21 found Jev's raw probabilities **overconfident**, with calibration dataset-dependent rather than a stable property: ECE 0.0204 on CLINC150 and 0.0936 on Banking77 in the same pre-registered study, `noul` ECE 0.169 and `choice` ECE 0.226 across 4,995 real agent decisions, and ECE 0.793 once an abstain option is removed. Type safety held perfectly — **zero type errors across 8,576 responses**. Full detail and caveats on [[independent-jev-calibration-audits-2026]].

TypeSafe's own jaggedness docs, reviewed 2026-09-17, concede that "jev-1.13's score levels are weak in numerical calibration" and that calibration is a **group-level** property, not a per-answer guarantee — which materially narrows what any single probability licenses you to do.

> [!note] What it is, by the founder's own account
> No architecture, parameter count or base model is public, and no paper exists — an arXiv search for "reinforcement learning for calibrated decisions" returns nothing. On Hacker News Diogo Almeida agreed that Jev is essentially a zero-shot classifier ("exactly right!") and that "Large Classification Model rather than a Large Language Model" was "very accurate!". He has said he will not publish ECE figures, considering public benchmarks gameable — which leaves the central claim unfalsifiable by the vendor and falsified-in-direction by everyone else.

Where frontier scale does show up is **label-space capacity**: Jev handles 255 options and takes Banking77's 77 intents at 0.870, where the 421M encoder [[laya]] collapses to 0.425.

## Since launch (to 2026-09-28)

- **No new model.** `jev-1.13.0` is still the only build; `jev-preview` aliases it. OpenRouter exposes a pinnable snapshot, `typesafe/jev-1.13-20260917`. TypeSafe advises pinning the exact version if you have tuned thresholds, because aliases move.
- **Signups paused** on 2026-09-22, citing demand. Latency incidents followed on 09-23 and 09-24.
- **Jev Router** launched on OpenRouter on 2026-09-25: a chat-request router that picks the model and reasoning effort per request and claims to be cache-aware. Its benchmark result (237 vs 130 of 423 tasks against OpenRouter's Auto Router) is vendor-reported.
- **Documented limits**: up to 255 options per Choice; Score takes 2–10 levels. OpenRouter's endpoint caps context at 32k tokens against 64k on TypeSafe's own.

> [!warning] Choice probabilities saturate; ask Noul per option instead
> A pre-registered audit found `choice` behaves close to a step function. With no evidence either way about a fair coin, it gives "heads" 0.83–0.93; told the base rate is 55/45, it gives 0.90–0.996. The bias follows the option *word*, not its key or position. `noul` tracked the exact targets within 0.015–0.027, and asking one `noul` per option and normalising cut Brier from 0.067 to 0.003. TypeSafe's jaggedness docs separately warn that Noul and Choice thresholds are not interchangeable. See [[independent-jev-calibration-audits-2026]].

**Security.** Typed output narrows [[prompt-injection]] without removing it: injected content in the state shifts decision probabilities, and adaptive attacks doubled success from 1.8% to 3.5% in the first paper on it. See [[jev-research-papers-september-2026]].

**Ranking.** On [[jevbench|JevBench]] Jev is 4th of 91 on the composite, behind three open 4B fine-tunes, but it leads the #3 system on the reasoning axis (Intelligence 53.1 vs 49.4) and on calibration. The open models win on speed and cost.

**As a judge.** [[jev-as-a-judge-accept-when-confident-li-2026|JEV-as-a-Judge]] (CMU, 2026) ranks Jev 8th of 15 judges, 1.7 points behind GPT-6 at 277× lower fee. It is well calibrated when a reference or pair anchors the verdict (ECE 0.01–0.07) and near chance without one. Escalating low-confidence verdicts to GPT-6 matched GPT-6's accuracy at 41% of its fee.

## Related

- [[typesafe-ai]] · [[system-one-model]] · [[calibration]] · [[logit-scoring]] · [[cloze-scoring]] · [[nimble]] · [[openjev]] · [[laya]] · [[independent-jev-calibration-audits-2026]] · [[jev-research-papers-september-2026]] · [[jevbench]] · [[kev]] · [[system-one-ecosystem-sweep-2026-09-28]]

## References

- [[introducing-system-one-models-and-jev-typesafe-2026]]

## Related 2026 sources

- [[jev-as-a-judge-accept-when-confident-li-2026]] — CMU pre-registered study of Jev as a cheap judge: well calibrated when a reference or pair anchors the verdict, near chance without one

## Lineage sources

- [[system-one-models-and-dual-process-thinking]] — how well 'System One' fits Kahneman's System 1
- [[s1-decision-model-raghavan-2026]] — open s1 matches Jev's ECE on JevBench (0.083 vs 0.081)
- [[microsoft-decision-1]] — Microsoft's competitor (Oct 2026)
- [[openai-decisions-api]] — OpenAI's competitor (Oct 2026)
- [[system-one-models-vs-system-one-thinking]] — name origin (Jevons)
