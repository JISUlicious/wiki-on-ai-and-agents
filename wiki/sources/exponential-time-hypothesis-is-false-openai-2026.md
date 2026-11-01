---
title: "The Exponential Time Hypothesis is False"
type: source
created: 2026-11-01
updated: 2026-11-01
sources:
  - exponential-time-hypothesis-is-false-openai-2026.md
authors:
  - OpenAI
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# The Exponential Time Hypothesis is False

**[[openai|OpenAI]]**, manuscript dated 25 Oct 2026, released 26 Oct in the [openai/math repository](https://github.com/openai/math). It was produced by an unnamed internal model and has no arXiv ID. Surfaced by the 31 Oct AI Agents Weekly. This page records the claim and its verification status. It does not assess the mathematics.

## Summary

The paper gives a deterministic algorithm that exactly counts the satisfying assignments of any k-CNF formula. For fixed k ≥ 2, n variables and m clauses, it uses polynomial space and runs in time (n+m)^O_k(1) · 2^(n/(log n)^(1+o(1))). That is subexponential, so if correct it refutes the Exponential Time Hypothesis, the conjecture that 3-SAT needs 2^Ω(n) time in the worst case. It does not settle P vs NP, since the bound is still superpolynomial.

## Key points

- **Wider than the title.** The paper states that its Theorem 1.1 refutes **ETH, SETH, Gap-ETH and NSETH**, plus bounded-error randomized versions of ETH, SETH and Gap-ETH. Many fine-grained lower bounds that assume these would lose that support.
- **Stated non-conflict.** It does not contradict unconditional exponential lower bounds for restricted proof systems such as regular resolution.
- **Part of a large release.** The repository holds 721 manuscripts in 373 families from about 4,000 posed problems, at about three hours of ChatGPT Pro thinking compute per result. About 42% of top-line results are formalized.
- **The repository's own framing.** OpenAI calls these manuscripts raw research data, says unformalized results "could have issues", defers to mathematicians on credit, and keeps a history that covers withdrawn papers.
- **Lean.** The 26 Oct history entry says this manuscript was added with an accompanying Lean formalization, and it is listed in the formalization catalogue. A check covers the formal statement, so people still have to confirm that the statement says what the paper claims.
- **Review.** Reporting says Impagliazzo and Paturi, who proposed ETH, and Ryan Williams will lead an independent examination, describe that as not an endorsement, and asked OpenAI to release raw data and fund a conference. OpenAI's page on this could not be fetched here (403), so this rests on [The Next Web](https://thenextweb.com/news/openai-exponential-time-hypothesis-false-kakeya-yang-mills) and on search excerpts of OpenAI's post.

> [!warning] Unverified as of this ingest
> At fetch time, no independent review outcome had been found. Treat "ETH is false" as a claim with a published formal proof under expert examination, not as a settled result. Some outlets also report a separate Anthropic refutation; that was seen only in secondary sources and is not recorded here as fact.

## What it does not show

- **Practical SAT gains.** It is a worst-case asymptotic bound, and the paper is not a solver study.
- **How autonomous the discovery was.** The model is unnamed and unreleased. Only 10 families have abridged reasoning summaries, and this manuscript is not among them.
- **The model's success rate on hard problems.** The catalog is filtered: outputs were aggregated and held to a significance bar, from about 4,000 attempts.

## Concepts & entities

- [[openai]]
- [[lean-formalization-cfsg-axiom-2026]]: a different kind of result from the same week, formalizing a known proof rather than finding a new one.
- [[leap-formal-mathematics-agentic-kung-2026]] · [[numina-lean-agent-liu-2026]]: agentic Lean proving.

## References

- [openai/math repository](https://github.com/openai/math)
- [Manuscript PDF](https://github.com/openai/math/blob/main/preprints/The-Exponential-Time-Hypothesis-is-False-October-25-2026/paper.pdf)
- [OpenAI: Working with academic communities](https://openai.com/index/building-scientific-understanding-with-academic-communities/)
