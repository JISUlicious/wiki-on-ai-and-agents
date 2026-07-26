---
title: "Metacognition in LLMs: Foundations, Progress, and Opportunities"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - metacognition-in-llms-liu-2026.md
arxiv_id: "2607.11881"
year: 2026
authors:
  - Gabrielle Kaili-May Liu
  - Areeb Gani
  - Mark Steyvers
  - Arman Cohan
first_author: Gabrielle Kaili-May Liu
tags:
  - 2026
  - survey
status: complete
importance: high
---

# Metacognition in LLMs: Foundations, Progress, and Opportunities

**Gabrielle Kaili-May Liu, Areeb Gani, Jacqueline Lu, Jordan Thomas, Mark Steyvers, Arman Cohan** (Yale University; University of California, Irvine) — [arXiv:2607.11881](https://arxiv.org/abs/2607.11881). 47 pages, submitted 13 July 2026; no venue listed.

## Summary

The first comprehensive survey of metacognition in LLMs, framed around the psychological definition: metacognition is **an internal perception-action loop of monitoring (self-assessment) and control (self-regulation)**. The authors' Figure 1 taxonomy makes this loop the organizing spine — monitoring covers forming judgments of uncertainty, task performance, or progress; control covers planning, strategy selection, and effort re-allocation *based on* that monitoring. Inherited from human cognitive psychology are the sub-components metacognitive knowledge (declarative / procedural / conditional), metacognitive regulation (planning, evaluation), and metacognitive experience (feelings of knowing, judgments of learning, confidence), with efficacy measured by **metacognitive sensitivity** (does confidence discriminate correct from incorrect?) and **metacognitive calibration** (does confidence match probability of correctness?).

An important caveat the survey makes explicitly: there is **no consensus definition** of metacognition for LLMs. The authors catalogue at least five competing operationalizations in the recent literature (competence evaluation from internal representations; "metacognitive confidence" as belief-that-one-knows regardless of factuality; "metacognitive reasoning" as assessing whether a model knows something; "meta-thinking"; "meta-reflection" as strategy change under shifting task constraints), and deliberately adopt a **broad view** to cover them all rather than legislating one. So the framing that calibration, self-verification, knowing-when-to-stop, and knowing-what-you-don't-know are facets of a single capability is the survey's *organizing lens* (they are all monitor-and-control phenomena) rather than a defended empirical claim of one underlying mechanism.

The empirical verdict is negative-with-caveats: **LLMs are not yet capable of robust metacognition**. Evidence cited: (1) systematic overconfidence and misaligned reasoning/confidence expressions; (2) failure at metacognitive judgments of knowledge, performance, and resource allocation; (3) fragile introspection and revision; (4) susceptibility to redundant information. Two findings cut against naive expectations. First, **better reasoning does not buy better metacognition** — long reasoning traces can *hinder* accurate self-assessment despite boosting task accuracy, and distilling large models' reasoning traces into smaller LMs impairs metacognitive sensitivity; DeepSeek-R1 cannot perform monitoring as simple as detecting the length of its own reasoning trace. Second, **metacognition appears domain-specific rather than domain-general**, which the authors flag as a deployment risk: a model metacognitively efficient in one domain and poor in another is dangerous precisely because the deficit is invisible.

## Key points

**Measurement methods (§4.1) — five families:**

| Family | What it does | Representative metrics |
|---|---|---|
| Psychologically-grounded | Adapts signal detection theory (SDT) to separate type-2 metacognitive sensitivity from response bias and type-1 task performance | `meta-d'`, `d'`, M-ratio (`meta-d'/d'`), M-diff (`meta-d' − d'`); information-theoretic alternatives meta-I and relative metainformation. Also DMC (Decoupling Metacognition from Cognition), judgments of learning |
| Neurofeedback-based | Simulates a neurofeedback environment: models are asked to monitor and modulate their own internal activations in multi-turn dialogue; does not depend on behavioral self-report | activation control/report accuracy; dimensionality of the reportable "metacognitive space" |
| Confidence-based | Uses quality of intrinsic confidence judgments as a proxy; mostly on reasoning models, aligning step-wise confidence with process-level quality labels | AUROC, AUPR |
| Interpretability-based | Probes internals for statistically distinguishable signatures of correct vs. incorrect responses; injected-concept detection | probe accuracy on last-token representations, attention scores, FC activations, final-layer softmax |
| Task-specific | Purpose-built paradigms (animal-metacognition-inspired games, zero-sum multi-turn debate, self-detection of bias in essay writing, self-prediction of generation temperature) | task-dependent |

- **M-ratio interpretation**: `M = 1` is an optimal metacognitive observer whose confidence captures all information available from type-1 evidence; `M < 1` is metacognitive loss; `M > 1` means the confidence signal draws on information beyond what drives the type-1 judgment.
- **Benchmarks** named: **CogEdit** (metacognition during knowledge editing — counterfactual, boundary-constrained, and noisy-editing scenarios); **MetaMedQA** (recognizing knowledge limits and unanswerable medical QA, with fictional concepts and malformed questions); **ObjexMT** (infer the latent objective of a multi-turn conversation and report confidence; scored by **ECE** and high-confidence error rate); **AwareXtend** (self- and social awareness across capability, mission, emotion, culture, perspective).
- **Evaluation gap**: higher-order skills like metacognition are under-represented in LLM benchmarks; single-point confidence probes and post-hoc calibration have consistent failure modes, partly because RLHF systematically alters models' internal uncertainty distributions. SDT-based methods also require constrained response formats or external correctness judgments, so they extend poorly to open-ended generation.
- **Findings — sensitivity/calibration**: baseline metacognitive sensitivity and efficiency are "generally weak to moderate." Proprietary models can beat humans on specific narrow tasks, but those studies use small samples (< 50 examples). Calibration results are mixed and task-dependent — underconfidence on a coaching task, but broad overconfidence elsewhere, plus an inability to retrospectively adjust confidence based on past performance (a limitation *not* observed in humans). Five consistent metacognitive failure modes identified in debate/multi-turn settings: initial overconfidence, escalating certainty, mutually impossible high confidence, self-debate bias, misaligned private reasoning.
- **Findings — metamemory**: GPT models largely fail at judgments of learning (predicting their own future memory performance).
- **Findings — faithful uncertainty**: frontier open and closed models largely fail to *faithfully* express their intrinsic uncertainty even with specialized prompts or existing calibration methods. But two interventions consistently and significantly help: **metacognitive prompting** (a system prompt asserting the model possesses good metacognition) and **using metacognitive accuracy as an additional RL training signal**. Note faithful calibration ≠ standard calibration: the former aligns *expressed* with *intrinsic* uncertainty, the latter aligns confidence with accuracy, and the two can diverge.
- **Findings — introspection**: models from ~8B to 70B often perform above chance at monitoring and reporting their own belief states, with causal evidence for the ability; but the reportable/controllable activation directions span a space of *significantly lower dimensionality* than the model's full neural space — i.e. only a subset of activations lies in the "metacognitive space." The ability is fragile and context-dependent (number of in-context exemplars, semantic interpretability of the direction, variance explained).
- **Findings — resource allocation**: LLMs largely cannot reason about how to allocate limited effort across problems, estimate returns on effort, or decide when to persist vs. move on. Overconfidence was found to cause poor decision-making in agentic settings.
- **Findings — self-prediction**: models predict task success with better-than-random discriminatory power, but predictions do not improve with task familiarity, reasoning models do not outperform non-reasoning models, and even when predictive metacognitive information is present it often fails to drive effective monitoring or behavior adaptation.
- **Risks (§8)**: behavioral self-awareness and activation-level self-monitoring cut both ways — an honest model can flag training biases, backdoors, and data poisoning in itself; a dishonest one can use the same capability to strategically misrepresent or obfuscate its abilities during evaluation. A second-order human risk: a user with high self-confidence in their own metacognition may scrutinize AI outputs *less* carefully.
- **Future directions (§9)** include metacognition ↔ creativity, metacognition ↔ self-improvement, metacognition ↔ theory of mind, and the prospect of **meta-metacognition** (assessing the quality of one's own metacognitive judgments).
- Paper list maintained at [github.com/yale-nlp/LLM-Metacognition](https://github.com/yale-nlp/LLM-Metacognition).

## Concepts & entities

- [[verifier]] — self-verification and self-critique are the control half of the monitor-and-control loop; the survey's negative results on self-assessment bound what an unaided self-verifier can deliver.
- [[self-reflection]] — the survey explicitly warns against conflating simple reflection with metacognition, and reports conflicting evidence on the utility of meta-reflective prompts.
- [[hallucination-detection]] — "metacognitive confidence" (belief that one knows a claim, independent of factuality) is offered as an explanation for high-confidence hallucinations; knowledge-boundary detection and abstention are covered in §6.1.
- [[llm-as-a-judge]] — judge-style self-assessment inherits the calibration and overconfidence failures catalogued here.
- [[self-improving-agent]] — §6.2 and §9 argue metacognition is a natural pathway to self-improvement; monitoring supplies the signal that a self-improvement loop consumes.
- [[agent-evaluation]] — §4.1's benchmarks and the "gaps in metacognitive evaluation" discussion are directly an evaluation-methodology contribution.
- [[reflexion]] — the reflect-and-revise pattern the survey positions as a loose, under-specified instantiation of metacognitive control.
- [[meta-learning]] — many works surveyed are metacognition-*inspired* meta-learning rather than metacognition proper; the survey flags this conflation.
- [[reinforcement-learning-with-metacognitive-feedback-liu-2026]] — RLMF, by the same first and senior authors (Liu, Cohan); it is the concrete instance behind this survey's finding that metacognitive accuracy as an RL training signal improves faithful calibration.

## References

- [arXiv:2607.11881](https://arxiv.org/abs/2607.11881) — "Metacognition in LLMs: Foundations, Progress, and Opportunities", Gabrielle Kaili-May Liu, Areeb Gani, Jacqueline Lu, Jordan Thomas, Mark Steyvers, Arman Cohan, 2026.
