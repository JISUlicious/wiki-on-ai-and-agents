---
title: "Mathematics in the age of AI"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.16753"
venue: Submitted to the Proceedings of the ICM 2026
authors:
  - Terence Tao
year: 2026
tags:
  - 2026
  - essay
  - ai-for-science
status: complete
importance: high
---

# Mathematics in the age of AI

**Terence Tao** (UCLA Department of Mathematics) — [arXiv:2608.16753](https://arxiv.org/abs/2608.16753), math.HO, submitted 17 Aug 2026 (v1; the paper's own title page is dated 24 August 2026). 12 pages, four figures. Comments field: *"Submitted to the Proceedings of the ICM 2026."* MSC: primary 00A30; secondary 01A80, 68T01, 68V20, 68V35. arXiv perpetual non-exclusive license.

> [!note] This is an essay, not an experiment
> This page summarizes a **position paper** — a written-up public lecture given at the International Congress of Mathematicians in July 2026. It reports no experiments of its own and makes no empirical claim about AI capability; it explicitly *conditions on* such capability and argues about values instead. The only quantitative figures below are ones Tao cites from other work. Nothing here should be read as measured evidence for or against what AI systems can do.

## The argument in one paragraph

Tao's move is a decomposition. The question "how should the mathematical community respond to AI?" contains a *capability* subquestion — what will these tools actually be able to do — which is where essentially all the public argument happens, and an *orthogonal* subquestion — what the goals and values of mathematical research actually are — which is where almost none of it happens. He sets the first aside by assumption and spends the essay on the second. The core claim is that mathematics has historically been able to leave its goals implicit because they were **positively correlated**, so that optimizing one or two of them served as a workable proxy for the rest; that heavy optimization pressure of the kind AI supplies breaks that correlation via Goodhart's law; and that the community therefore has to make its unwritten goals explicit — exactly as the early-twentieth-century crisis forced it to make its unwritten *foundations* explicit. He works this out on one case study, problem solving, and finds that a goal which looks like a single arrow is really a five-stage pipeline of which **only the first stage was ever an explicit community goal**.

## Structure of the argument

### 1. The historical analogy: a second crisis, of values rather than truth

For centuries mathematics ran on "naive" foundations: working mathematicians proved theorems about sets, numbers and infinities without saying precisely what those were, or what a proof was, delegating such questions to philosophers. Russell's paradox (1901) and Gödel's incompleteness theorems (1931) forced that implicit layer into the open. The resulting **crisis in foundations, roughly 1900–1930, was genuinely turbulent** — but its end product was "an explicit, rigorous, and standardized foundational framework, in which the objects of mathematics and the rules for reasoning about them are laid out in a form that can be inspected, taught, and — as it turns out — mechanized."

The analogy is precise about what is *different* this time. What is now being stress-tested is **not the framework for mathematical truth but the largely implicit framework of mathematical values and practices**: "what we consider a contribution to be, what we reward, what we regard as understood, and who — or what — we regard as having done the work." The prediction attached to the analogy is optimistic and conditional: making these goals explicit will be turbulent, and the community "will emerge stronger and more resilient than before."

Note the reflexive sting in the prologue: the last crisis produced foundations rigorous enough to be *mechanized*, which is a precondition for the machines now causing the second crisis.

### 2. The decomposition

**Question 2.1 (Community Response Question).** *How should the mathematical community respond to the advent of modern AI technologies, and their real and/or claimed capabilities to perform mathematical tasks?*

Tao is careful that this "is not a mathematical question. It is a metamathematical one, and also a political, ethical, sociological, and cultural one; it cannot be settled by pure logical argument." He then borrows the *form* of mathematical language — conjecture, hypothesis, template — purely to expose the structure.

**Conjecture 3.1 (AI Capability Conjecture, template form).** *"At some point in the near future, some AI tools will, at some expense, and with some level of human supervision, be able to accomplish some research-level mathematical tasks in some fields of mathematics, with some non-trivial success rate, and at some level of correctness and quality."* Every "some" is a free parameter, so this is a *family* of conjectures, not one; he only distinguishes coarsely between weak and strong forms. If even weak forms are false, business as usual; if the strongest forms hold, current culture is very hard to maintain unchanged — "particularly if we continue to prioritize such goals as obtaining as many solutions to unsolved problems as possible."

His diagnosis of why the debate is stuck is sharp and directly relevant to how this wiki treats agent claims. Most data points bearing on the conjecture "have not been gathered under controlled scientific conditions"; the public evidence suffers **severe reporting bias** (successes announced, failures not) and non-scientific incentives, with key variables "frequently left undisclosed" — he names four: *the number of attempts, the amount of human scaffolding, the compute expended, and the degree of contamination of the problem with prior literature*. And, separately, "the truth value of a given form of the conjecture is sometimes conflated with its desirability."

The one capability result he does cite, as an example of controlled evaluation, is the **First Proof project**. Each batch is ten research-level problems contributed by working mathematicians, with solutions known to the contributor but never posted anywhere online. The second batch was evaluated against four AI systems using models publicly accessible as of **28 May 2026**, and the solutions were refereed by experts for correctness *and* quality of exposition. **Seven of the ten problems received at least one passing grade** — a solution judged essentially flawless or needing only minor revisions — **from at least one system**, at compute costs "on the order of tens to hundreds of dollars per problem." He reports this and moves on; the essay is emphatically not about the conjecture.

**Hypothesis 4.1 (Working Hypothesis).** *"A reasonably strong version of the AI Capability Conjecture is true: AI tools will, reasonably soon, become capable of performing a reasonable fraction of research-level mathematical tasks, with reasonable levels of success, quality, supervision, and cost."* The framing of what he is asking of the reader is worth quoting exactly, because it is what keeps the essay from being a capability prediction: *"I am not asking the reader to want it to be true, to believe that it is true, or to accept it as true; what follows is a conditional analysis."*

### 3. The orthogonal subquestion, and why the goals come apart

**Question 5.1 (Goals and Values Question)** asks for the community's real objectives — "not merely the explicit goals that we communicate to the public, to our students, or to funding agencies, but the implicit goals that we actually optimize for in practice." He notes that mathematics has historically delegated this question to historians, philosophers and sociologists of mathematics, and that under the Working Hypothesis "we will no longer have this luxury."

His deliberately partial list of goals (readers are invited to extend it):

- solve unsolved problems, pure and applied;
- develop new theories, structures, and techniques;
- understand the world around us;
- build and sustain a community of mathematicians;
- train the next generation, and let them guide the future directions of the subject;
- contribute to the shared and cumulative network of mathematical knowledge;
- create enduring works of aesthetic value.

The load-bearing observation is that these have been **positively correlated**: solving a hard problem tends to produce a technique, around which a community forms, students are trained, textbooks written, and the theory turns out to apply elsewhere. Because of that correlation, "one could use one or two of these goals as convenient proxies for the others" and leave the rest unstated (Figure 1; he cites his own 2007 *What is good mathematics?*).

**Goodhart's law** is the mechanism that breaks it — in Strathern's formulation, *"When a measure becomes a target, it ceases to be a good measure."* Tao gives two independent reasons AI is unusually likely to trigger it:

| | Reason | Why it bites |
|---|---|---|
| **Technical** | Generative AI is **inherently ungrounded** | It "optimizes for the appearance of a satisfactory output rather than for the underlying property that the output is supposed to indicate, and so is unusually good at finding the gap between a measure and the thing it measures" |
| **Economic** | Industry incentives | "The financial incentives of the AI industry reward demonstrable, quotable, benchmarkable achievement on precisely the sort of metrics we have historically used as proxies" |

Figure 2 is the payoff: under excessive optimization the previously aligned goals **diverge**, with "solve Erdős problems" and "solve Olympiad problems" splitting off from "build theory", "build community", "train the next generation", "create enduring aesthetic works". He flags the diagram as "extremely oversimplified; in particular, it should be very much higher dimensional."

### 4. The case study: problem solving as a five-stage pipeline

Tao chooses problem solving as a first case study for two stated reasons — it is the aspect most susceptible under the Working Hypothesis, and "the aspect for which our implicit goals are the furthest from our explicit ones" — while stressing that **theory building is "a complementary activity of at least equal significance"** requiring its own analysis, as do teaching, mentoring, and service.

The essay's method is to write the goal down, find the failure mode, and rewrite it. Five iterations:

| Goal | Statement | The failure mode that forces the next revision |
|---|---|---|
| 6.1 | Solve as many unsolved problems as possible | Produces a flood of *incorrect* solutions — a pre-AI, entirely mundane failure, familiar to anyone with a public email address and a steady stream of purported proofs of the Riemann hypothesis |
| 6.2 | …and verify them to be correct | A verified proof **nobody understands**. Not hypothetical: the Erdős problems database "already contain[s] dozens of AI-generated proof submissions", many likely correct, but in a substantial number no human expert has volunteered to vouch for them and "in several cases the human submitters have themselves declared that they are not qualified to do so" |
| 6.3 | …and ensure the results can be clearly communicated to and understood by the community | Exposition can itself be **over-optimized** — see below |
| 6.4 | …and have them digested and accepted by the community | Acceptance is "slow and human" and "ultimately an external process that cannot be optimized purely by the authors and their tools"; the volunteer referee system is the bottleneck |
| 6.5 | …and incorporated into the definitive theory of the field | (the final form; canonicalization is the slowest and, he argues, most valuable stage) |

Figure 4's caption states the punchline: *"What began as a single arrow has become a chain of five stages, of which only the first was ever an explicit goal of the community."* The pipeline runs open problems → unverified solutions → verified solutions → well-written solutions → accepted solutions → definitive solutions, via proof generation, verification, exposition, publication/digestion, and canonicalization.

Three arguments inside this ladder carry most of the essay's weight:

**Verification without understanding.** On formalization (Rocq, HOL, Lean, Mathlib) he is straightforwardly positive — "a formally verified proof is, after all, precisely a proof whose correctness no longer depends on the reputation or the diligence of its author" — which is exactly why it *does not* solve the problem: correctness and comprehension come apart, and "we may soon be faced with the very real possibility of a verified proof of a major result that no human understands well enough to explain."

**Over-polished exposition destroys tacit knowledge.** This is the essay's most distinctive point and the least likely to be anticipated. Current AI writing has near-flawless spelling, grammar and formatting (footnote: "One can argue that they are too flawless") but "very often dwells at length on trivialities while passing briefly through — or even actively obscuring — the most interesting and novel portions of the argument", and fails to situate results in prior literature or supply the high-level overview a reader needs to decide whether to invest. He expects the Working Hypothesis to fix much of that — and then argues that fixing it too well is also a failure. A human proof retains **natural friction** where the author struggled: "an apologetic remark, an unusually careful lemma, a change of notation, a paragraph that has clearly been rewritten several times." That friction "signals to the reader where to slow down and pay attention, and it is one of the main channels by which the tacit knowledge of a field is transmitted." Excessive polish sands away both the artificial friction (typos, awkward phrasing, disorganization) and the natural kind, "leaving a text that is easy to read and hard to learn from." Figure 3 is a page of Bourgain's 1991 paper annotated by a young and "very frustrated" Tao — the point being that fighting through it is how he came to understand Bourgain's way of thinking. He anchors this to Thurston (1994): *"We are not trying to meet some abstract production quota of definitions, theorems and proofs. The measure of our success is whether what we do enables people to understand and think more clearly and effectively about math."*

**Acceptance and canonicalization are the parts that cannot be automated.** Authors can assist digestion "by describing the insights, the false starts, and the stories from the period when they were working on the problem" — and "current AI tools are quite opaque about their own problem-solving process, and this is particularly true of proprietary models whose inner workings are a corporate secret." The refereeing labour that produces acceptance is volunteer, largely uncredited, and "routinely regarded as less prestigious than the work of generating proofs in the first place", yet "it is precisely the mechanism by which the individual achievements of mathematicians are converted into collective progress and understanding." On automated review he takes a specific, limited position: AI evaluators "may well serve as useful filters", letting journals triage submissions flagged for inadequate verification, missing attribution or incoherent exposition much as plagiarism detection works today, thereby conserving "the scarce resource of expert human attention" — **but** "passing an automatic filter is not a substitute for community acceptance; I do not believe that human referees can be removed from the publication process." Canonicalization — a result "restated in its natural generality, given its right proof rather than its first proof, connected to its neighbors, and absorbed into the standard toolkit" — is slowest, least automatable, and in his view most valuable. And it closes the loop: "the training data for these tools is, quite literally, the output of the canonicalization process."

### 5. Proof scarcity → proof abundance

If the Working Hypothesis holds and nothing changes, "impedance mismatches" — his less flattering metaphor is **proof indigestion** — appear at every stage:

- AI-generated proofs accumulate faster than they can be verified;
- verified AI proofs accumulate faster than they can be given a readable write-up;
- proofs that are both correct *and* well written will overwhelm a peer review system that depends on volunteer expert labour;
- even published proofs will be too numerous for the community to work into definitive form.

The institutional claim: journals, priority conventions, hiring and promotion criteria, prizes, "the very notion of a research program" were all designed under scarcity and "it should not surprise us if they behave poorly under abundance." He is careful to note the strain predates AI — growing literature volume, increasing length and specialization of major proofs, documented pressure on refereeing — and that AI "will exacerbate these existing stresses markedly."

## What he actually recommends

Less than one might expect, and deliberately so: *"I do not intend to propose a full program here."* The essay's own contribution is the diagnostic frame — once each mismatch is attached to an identified stage and an identified value, the response becomes tractable. For the program itself he defers to the **Leiden Declaration on Artificial Intelligence and Mathematics** (published 2 June 2026, endorsed by the International Mathematical Union, arising from a 2025 workshop at the Lorentz Center in Leiden): **23 recommendations** addressed to individual mathematicians, to mathematical organizations and not-for-profit funders, and to policymakers. He quotes four of the recommendations to individuals, plus a fifth in his closing, and comments on each:

| Leiden recommendation | Tao's commentary |
|---|---|
| **Disclose tool use** — include a "Tool and computational resource disclosure" section; live up to the spirit of the UNESCO Recommendation on Open Science and the FAIR principles | "The scenario to be avoided at all costs is one in which authors use AI tools covertly to aid their work, but conceal that usage in order to avoid criticism from their peers" |
| **Support the needs of reviewing** — disclose tool use, give precise and complete references, provide formal proofs where feasible | "We need to **decrease** the emphasis that our culture places on proof generation, and in particular on being the 'first' to solve a problem, and correspondingly **increase** the emphasis we place on proof digestion: exposition, refereeing, publication, and canonicalization" (citing Bessis on "the fall of the theorem economy") |
| **Affirm the humanity of authorship** — credit and responsibility belong to humans, not automated systems | quoted without dissent |
| **Put effort into proper attribution** — the tools' known attribution failures create a corresponding obligation; where satisfactory attribution is impossible, say so explicitly | quoted without dissent |
| **Participate in public discourse** (quoted in §9) | mathematicians must contextualize AI-assisted results, "particularly … within our own subfields, where specialized knowledge is required to assess claims about the depth, difficulty, and significance of results" |

His one original, concrete proposal is a publication norm, stated as a personal rule of thumb:

> *"If the authors cannot convincingly demonstrate that they are able to give a clear, expert-level talk on their results, one that is correct and properly attributed, then the result should not be published. A proof that no human can properly explain should be viewed as incomplete, even if it has been formally verified."*

The closing section generalizes the method rather than the conclusions. The same goal-decomposition should be run on teaching, mentoring, hiring, grant applications, refereeing and public outreach, and **"the conclusions of such analyses will not be uniform."** In education he is restrictive — emphasize the irreducibly human, restrict AI tools "quite tightly", because "the goal of training a mathematician is not achieved by producing correct homework." Elsewhere he wants the community on the front foot, defining best practice "on our own terms rather than on terms set for us by vendors", and building new infrastructure alongside the traditional kind: collaborative formalization projects, structured problem databases, and new venues for exposition and for **publishing negative or partial results**. Appendix A lists live examples — Mathlib, Mathematical Discourse (a peer-reviewed video journal for research talks), the Erdős problems database, an optimization constants database, the SAIR Foundation competitions, the First Proof project, and the Palomar formalized proof registry.

He practices the first recommendation on himself. The acknowledgments disclose: *"AI assistance was used to perform literature search, to generate diagrams, to autocomplete text, and to convert the slides into a paper format."*

## What this essay does *not* claim

Worth being explicit, since the title invites over-reading:

- **No capability claim.** The Working Hypothesis is assumed, not argued; the essay is a conditional analysis and says so three separate times. The First Proof numbers (7/10, four systems, tens-to-hundreds of dollars per problem) are cited from a preprint, not produced here.
- **No timeline.** "At some point in the near future" is explicitly a free parameter.
- **No claim that AI cannot do exposition or acceptance.** He predicts AI "will improve at [exposition] considerably from current levels"; his argument is that *even successful* exposition can be over-optimized, and that acceptance and canonicalization are structurally community processes rather than tasks with an owner.
- **No general anti-AI position.** He supports formalization, endorses AI filters for editorial triage, discloses his own AI use, and calls for mathematicians to define AI workflows rather than refuse them.
- **No full policy program.** He points to the Leiden Declaration and offers one rule of thumb of his own.
- **One place where his own framing limits the scope:** the case study is problem solving only. Theory building, which he calls of at least equal significance, is left for future analysis.

## Related

- [[reward-hacking]] — Goodhart's law is the essay's central mechanism, and Tao's technical reason ("generative AI is inherently ungrounded … unusually good at finding the gap between a measure and the thing it measures") is a compact statement of the specification-gaming argument in a non-ML register.
- [[hallucination]] — the ungroundedness claim, and the concrete instance of formally-verified-but-unexplained proofs accumulating in the Erdős database: correctness and comprehension coming apart.
- [[verifier]] — formal verification is Tao's Goal 6.2 and he is positive about it, but the essay's whole point is that a verifier discharges *one* stage of a five-stage pipeline. Useful counterweight to reading verifiability as the terminal goal.
- [[llm-as-a-judge]] / [[reliability-without-validity-llm-as-a-judge-norman-2026]] — Tao's carefully bounded position on automated review (useful triage filters, never a substitute for human referees) is the humanities-side statement of the reliability-vs-validity distinction.
- [[agent-benchmark]] / [[agent-evaluation]] / [[beyond-the-leaderboard-albayaydh-2026]] — his critique of AI-mathematics evidence is a benchmark-methodology critique: reporting bias, undisclosed attempts, undisclosed scaffolding, undisclosed compute, contamination.
- [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] / [[harness-vs-model-attribution]] — "the amount of human scaffolding" is item two on Tao's list of undisclosed variables. Independent arrival at the wiki's harness-disclosure position, from a Fields Medalist with no stake in agent benchmarking.
- [[leap-formal-mathematics-agentic-kung-2026]] / [[numina-lean-agent-liu-2026]] — the Lean/formal-mathematics agent work that sits at Goal 6.2 of the pipeline; Tao's argument is about the four stages downstream of them.
- [[training-ai-scientists-replicate-research-falck-2026]] — the same "our evaluation targets are proxies for what we actually want" problem, addressed by construction rather than by argument: a rubric judge deliberately scoring process and integrity rather than output fidelity.
- [[accelerating-scientific-research-gemini-real-world-schmidgall-2026]] — an AI-for-science pipeline that explicitly ends at paper *generation*; Tao's Figure 4 argues that generation is the first of five stages and the only one that was ever an explicit goal.
- [[frontier-lms-struggle-to-copy-wen-2026]] — evidence bearing on the capability subquestion Tao brackets.
- [[test-time-compute]] — First Proof's "tens to hundreds of dollars per problem" is the cost axis of the capability conjecture, one of Tao's four routinely undisclosed variables.

**Concept and entity gaps noted, not created:** `terence-tao` (person entity — a Fields Medalist with no page despite this essay and *What is good mathematics?*), `ai-for-science` (used as a tag here and on two other 2026 sources with no concept page), `formal-verification` / `proof-assistant` / `lean` (Mathlib, Lean, Rocq, HOL are all referenced and none has a page), `goodharts-law` (arguably distinct from [[reward-hacking]] and cited across several wiki pages), `ucla` (university entity).

## References

- [arXiv:2608.16753](https://arxiv.org/abs/2608.16753) — Terence Tao, *Mathematics in the age of AI*, math.HO, 17 Aug 2026. Submitted to the Proceedings of the ICM 2026; based on a public lecture at the ICM, July 2026.
- The Leiden Declaration on Artificial Intelligence and Mathematics, 2 June 2026 — 23 recommendations, endorsed by the International Mathematical Union.
- M. Abouzaid, N. Srivastava et al., *First Proof Second Batch*, arXiv:2606.18119 — the controlled capability evaluation Tao cites.
- W. P. Thurston, *On proof and progress in mathematics*, Bull. AMS 30 (1994) — the essay's touchstone.
- T. Tao, *What is good mathematics?*, Bull. AMS 44 (2007) — his earlier treatment of the goal-alignment phenomenon.
