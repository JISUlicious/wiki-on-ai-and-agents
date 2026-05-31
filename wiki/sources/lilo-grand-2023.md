---
title: "LILO: Learning Interpretable Libraries by Compressing and Documenting Code"
type: source
created: 2026-05-29
updated: 2026-05-29
sources:
  - lilo-grand-2023.md
arxiv_id: "2310.19791"
venue: ICLR 2024
authors:
  - Gabriel Grand
  - Lionel Wong
  - Matthew Bowers
  - "[[josh-tenenbaum]]"
  - "[[jacob-andreas]]"
first_author: Gabriel Grand
year: 2023
introduces:
  - "[[library-learning]]"
tags:
  - 2023
status: complete
importance: medium
---

# LILO: Learning Interpretable Libraries by Compressing and Documenting Code

**arXiv:** [arXiv:2310.19791](https://arxiv.org/abs/2310.19791) · **Venue:** ICLR 2024 · **Year:** 2023
**Authors:** Gabriel Grand, Lionel Wong, Matthew (Maddy) Bowers, Theo X. Olausson, Muxin Liu, [[josh-tenenbaum|Joshua B. Tenenbaum]], [[jacob-andreas|Jacob Andreas]] ([[mit|MIT CSAIL / BCS]], Harvey Mudd College)

## Summary

**LILO** (Library Induction from Language Observations) is a neurosymbolic framework that iteratively *synthesizes, compresses, and documents* code to build libraries of reusable function abstractions tailored to a problem domain. It is the direct successor to [[dreamcoder|DreamCoder]] (Ellis et al. 2021), the prior state-of-the-art [[library-learning]] system. DreamCoder used a Wake-Sleep loop — search for task solutions (Wake), then refactor shared structure into a library (Sleep) — but its enumerative search was crushingly expensive (>2 CPU-months per domain) and its learned λ-abstractions were opaque, decipherable only with lambda-calculus expertise. LILO replaces the neural recognition model with an LLM and the bespoke compression step with a faster symbolic algorithm, then adds a step DreamCoder lacked entirely: natural-language documentation of every abstraction.

The architecture is a three-module loop. (1) A **dual-system synthesis** module solves tasks two ways — *LLM-guided search* (querying Codex `code-davinci-002` with the NL task description plus the current library, giving strong domain-general priors) and *enumerative search* over the DSL (which discovers domain-specific structures, e.g. how to draw a "snowflake," that language alone cannot infer). (2) A **compression** module runs [[dreamcoder|STITCH]] (Bowers et al. 2023), a branch-and-bound symbolic compressor that finds optimal λ-abstractions of minimal description length across the whole solution corpus — 1000–10000× faster and 100× more memory-efficient than DreamCoder's refactorer, fast enough to re-derive the entire library from scratch every iteration ("deep refactoring"). (3) An **auto-documentation (AutoDoc)** module sequentially prompts an instruction-tuned LLM (`gpt-3.5-turbo`) to assign each anonymous STITCH abstraction (`fn_42`) a human-readable name and docstring (`vowel_regex`, "matches any vowel…"), with names cascading so later abstractions reference earlier ones (`consonant` = `(not vowel_regex)`).

LILO's central methodological contribution to the wiki's "skill" vocabulary is that it gives a **rigorous, formal definition of an abstraction/skill**: an optimal λ-abstraction extracted by minimizing description length over a program corpus, equipped with an inferred NL name and docstring. This is the program-synthesis grounding of what [[voyager]] and the [[skill-md-format|SKILL.md]] convention treat operationally — Voyager's "skill" is just any code function the agent happened to write and index by description, with no optimality or minimality guarantee. LILO instead *derives* the abstraction as the compression-optimal refactoring of solved tasks, then documents it. So LILO supplies the formal ancestor for the looser modern notion of an "[[agent-skills|agent skill]]": a reusable, named, documented unit of executable behavior that an LLM can compose. AutoDoc is also load-bearing, not cosmetic — naively handing the LLM anonymous STITCH abstractions *hurt* solve rates (e.g. −30.6 pts on REGEX, a form of code obfuscation); documenting them recovered and exceeded baseline. This is a concrete instance of the [[code-as-harness]] thesis that legible, named code is what makes a learned library usable by an LLM.

## Key Points

- **Three-module search-compress-document loop** (Alg. 1): per iteration, run LLM Solver → run enumerative SEARCH → COMPRESS with STITCH to mint top-k abstractions → REWRITE all solutions against the new library → AutoDoc each new abstraction. The library is rebuilt from the base DSL `L₀` every iteration, letting LILO discard suboptimal early abstractions.
- **Dual-system synthesis.** LLM-guided search supplies domain-general priors and sample efficiency; enumerative search finds domain-specific structure that language cannot express. They are complementary, and the full LILO (both) wins; one round of LLM synthesis is estimated equivalent in dollar cost to ~48 CPU-hours of DreamCoder search.
- **STITCH symbolic compression.** Refactoring is cast as a description-length minimization (Eq. 4): find abstraction `f` minimizing `|f| + Σ |REWRITE(π)|`. STITCH solves this via branch-and-bound, typically in seconds on one CPU — the efficiency that makes per-iteration deep refactoring tractable.
- **AutoDoc is not optional.** Anonymous abstractions act as code obfuscation and *degrade* LLM synthesis (−30.60 REGEX, −2.91 CLEVR, −11.11 LOGO vs. LLM-only). Adding NL names + docstrings recovers it (+9.73 REGEX, +2.27 CLEVR). Naming cascades sequentially so the library reads coherently.
- **An abstraction = a refactored, reusable, optimally-compressed, documented function** — a rigorous definition, in contrast to the operational "any indexed code function" notion in [[voyager|Voyager]]/[[skill-md-format|SKILL.md]].
- **Contrast with DreamCoder's Wake-Sleep:** LILO keeps the iterative search↔library bootstrapping idea but swaps (a) DreamCoder's neural recognition model + enumeration for LLM-guided search, (b) DreamCoder's slow compression for STITCH, and (c) adds AutoDoc. Result: tractable cost and interpretable libraries.
- **Results (online synthesis, mean solve-rate Δ vs. DreamCoder):** REGEX **+33.14**, LOGO **+20.42**, CLEVR +2.26 (CLEVR baseline already ~94.5%). LILO also beats an LLM-only solver (+0.94 REGEX, +8.09 CLEVR, +16.82 LOGO). Famously, LILO learns the concept of a *vowel* — a stepping stone that pure enumeration would need to find among 26⁵ disjunctions of character primitives.
- **Libraries generalize without language.** In offline synthesis (frozen library, enumerative search, no NL), LILO's final library `Lf` improves over the base DSL by +42.27 (REGEX), +63.43 (CLEVR), +43.24 (LOGO), beating both DreamCoder and post-hoc-compressed LLM libraries — confirming compression+documentation *in the loop* yields higher-quality libraries.
- **Honest limitations.** AutoDoc sometimes produces uninformative, ambiguous, or incorrect docstrings (e.g. LOGO polygon functions named by implementation not behavior; a "doubling length" claim that is wrong); because naming is sequential it can "double down" on a wrong description. Authors suggest self-consistency / verification as future fixes. LILO's DSLs are lambda-calculus; extending to imperative languages like Python is framed as tractable PL work.

## Entities

- [[mit|MIT]] — MIT CSAIL and Brain & Cognitive Sciences; institutional home of the work.
- [[josh-tenenbaum|Joshua B. Tenenbaum]] — co-author; long line of program-induction / Bayesian models of concept learning.
- [[jacob-andreas|Jacob Andreas]] — co-author; language-and-program work (LAPS, the source of the three benchmark domains).
- Gabriel Grand — first author (corresponding); Lionel Wong, Maddy Bowers (STITCH), Theo X. Olausson, Muxin Liu — co-authors.

## Concepts

- [[library-learning]] — the core task LILO addresses; iteratively inducing a domain library of reusable abstractions.
- [[dreamcoder|DreamCoder / STITCH]] — DreamCoder is the direct ancestor (Wake-Sleep library learning); STITCH is the symbolic compressor LILO adopts.
- [[skill-library]] — LILO's documented λ-abstraction library is the program-synthesis analogue; cf. Voyager's skill library.
- [[agent-skills]] — LILO gives a formal grounding for the "named, documented, reusable unit of behavior" that agent-skill frameworks treat informally.
- [[code-as-harness]] — AutoDoc's finding (legible named code is what makes a library usable to an LLM) is a direct instance of this thesis.
- [[voyager]] — comparison point: Voyager's skills are operationally defined; LILO's are compression-optimal and formally specified.
- Related: [[chain-of-thought]], [[self-consistency]] (proposed for improving AutoDoc), [[in-context-learning]] (the "transformer attention as non-compressive library learning" remark).

## Quotes

> "While large language models (LLMs) now excel at code generation, a key aspect of software development is the art of refactoring: consolidating code into libraries of reusable and readable programs." — Abstract

> "To make these abstractions interpretable, we introduce an auto-documentation (AutoDoc) procedure that infers natural language names and docstrings based on contextual examples of usage. In addition to improving human readability, we find that AutoDoc boosts performance by helping LILO's synthesizer to interpret and deploy learned abstractions." — Abstract

> "DreamCoder's search procedure is extremely computationally intensive, requiring more than two CPU-months just to learn a single domain… Moreover, DreamCoder libraries are not necessarily interpretable, requiring both domain expertise and knowledge of lambda calculus to decipher." — §1

> "We hypothesized that rewriting program examples in terms of these anonymous lambdas during the compression step was acting as a form of code obfuscation." — §3 (motivating AutoDoc)

> "In contrast to the view that LLMs will subsume formal accounts of programming languages, LILO offers a blueprint for collaboration between the ML and PL communities towards the longstanding goal of learning interpretable software libraries that enable solutions to novel problem domains." — §5

## References

- Grand, Wong, Bowers, Olausson, Liu, Tenenbaum, Andreas (2024). *LILO: Learning Interpretable Libraries by Compressing and Documenting Code.* ICLR 2024. [arXiv:2310.19791](https://arxiv.org/abs/2310.19791). Code: github.com/gabegrand/lilo
- Ellis et al. (2021). *DreamCoder: Bootstrapping inductive program synthesis with wake-sleep library learning.* — direct ancestor.
- Bowers et al. (2023). *Top-down synthesis for library learning (STITCH).* POPL 2023 — the symbolic compression engine.
- Wong et al. (2021). *Leveraging Language to Learn Program Abstractions and Search Heuristics (LAPS).* — source of the REGEX / CLEVR / LOGO domains.
- Wang et al. (2023). *Voyager* — embodied programmatic skill libraries, cited as related work; cf. [[voyager-wang-2023]].
