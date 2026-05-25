---
title: "PAL: Program-aided Language Models"
type: source
created: 2026-05-25
updated: 2026-05-25
sources:
  - pal-gao-2023.md
arxiv_id: "2211.10435"
authors:
  - Luyu Gao
  - Aman Madaan
  - Shuyan Zhou
  - Uri Alon
  - Pengfei Liu
  - Yiming Yang
  - Jamie Callan
  - "[[graham-neubig]]"
first_author: Luyu Gao
year: 2023
introduces:
  - "[[pal]]"
  - "[[program-aided-reasoning]]"
tags:
  - 2023
status: complete
importance: high
---

# PAL: Program-aided Language Models

[arXiv:2211.10435](https://arxiv.org/abs/2211.10435) · Luyu Gao, Aman Madaan, Shuyan Zhou, Uri Alon, Pengfei Liu, Yiming Yang, Jamie Callan, [[graham-neubig]] ([[cmu]]) · 2023 (ICML 2023)

## Summary

PAL (Program-Aided Language models) is a prompting method in which an LLM reads a natural-language problem and emits a **Python program** as its intermediate reasoning chain; the program is then executed by a standard Python interpreter, and the interpreter's output is taken as the final answer. The LLM is responsible only for *decomposition* (turning the problem into runnable code with meaningful variable names and NL comments); *solving* — arithmetic, lookups, loops, dictionary aggregation — is fully offloaded to the interpreter. This bridges the well-known failure mode of [[chain-of-thought-prompting]] in which the natural-language reasoning chain is correct but the model still produces an incorrect numeric answer.

PAL is the canonical origin of "code as the reasoning substrate" — the [[code-as-harness]] survey traces its §2.1.1 *Program-Delegated Reasoning* branch back to this paper. Empirically, PAL with Codex (`code-davinci-002`) sets few-shot SOTA on 13 mathematical, symbolic, and algorithmic benchmarks: on GSM8K it beats PaLM-540B+CoT by **+15% absolute** (72.0 vs. 56.9), and on the authors' GSM-HARD variant (numbers replaced with up to 7-digit integers) PAL holds at 61.2% while CoT collapses from 65.6 to 20.1 — a +40% absolute gap. A smaller backbone using a symbolic interpreter beats a much larger pure-text model.

## Key Points

- **Mechanism**: in-context examples pair each NL problem `x` with a *mixed NL+PL* trace `t` whose NL fragments are written as Python comments and whose PL fragments compute the answer; the LLM completes a fresh problem in the same format, the trace is post-hoc executed by a Python interpreter, and the value of `answer` is returned as `y`. No final-answer label is given in the prompt — solving is delegated entirely to the runtime.
- **Critical design choices**: (1) meaningful, semantically-grounded variable names (`num_apples_in_basket`, `loaves_baked`, `stress_ball_idx`) tie code tokens to entities in the question; §6 ablations show this is critical, not cosmetic. (2) NL steps as comments help on harder symbolic tasks but are unnecessary on plain math word problems.
- **Math reasoning (Table 1, GSM8K solve-rate)**: Direct Codex 19.7 · CoT UL2-20B 4.1 · CoT LaMDA-137B 17.1 · CoT Codex 65.6 · CoT PaLM-540B 56.9 · CoT Minerva 540B (math-finetuned) 58.8 · **PAL Codex 72.0**. PAL wins on all eight math datasets (GSM8K, SVAMP, ASDIV, SINGLEEQ, SINGLEOP, ADDSUB, MULTIARITH, GSM-HARD).
- **GSM-HARD**: the authors' robustness benchmark replaces a number in each GSM8K problem with a random up-to-7-digit integer. Direct drops 19.7→5.0 (-74% relative); CoT drops 65.6→20.1 (-70%); PAL drops 74.0→61.2 (only -14%). Demonstrates that CoT's "reasoning" is partially memorized small-number arithmetic.
- **Symbolic/algorithmic (Table 2)**: PAL beats CoT Codex by large margins on COLORED-OBJECTS (95.1 vs. 86.3), PENGUINS (93.3 vs. 79.2), DATE (76.2 vs. 64.8), REPEAT-COPY (90.6 vs. 68.8), and OBJECT-COUNTING (96.7 vs. 73.0).
- **Eliminates arithmetic errors entirely**: any failure of PAL is a *decomposition* failure, not a calculation failure. This separates the two error modes CoT conflates and is the core conceptual contribution.
- **Decoder choice**: code-finetuned LLMs (Codex `code-davinci-002`) outperform natural-language LLMs as the PAL backbone — generating Python is in-distribution. PAL also works with non-code LLMs but with smaller gains.
- **Cousin / contemporaneous work**: Program-of-Thoughts (PoT, Chen et al. 2022) proposes essentially the same delegate-to-interpreter idea concurrently; PAL and PoT are typically cited together as co-origins of program-delegated reasoning.
- **Extends to Least-to-Most (Appendix I)**: PAL composes with question-decomposition prompting strategies, not just CoT.
- **Author's framing**: "an essential step towards general and robust AI reasoners" — neural model for language understanding, symbolic system for execution; an explicit endorsement of neuro-symbolic hybridization.

## Related Entities

- [[graham-neubig]] — senior author; runs the [[cmu]] LTI group that produced this paper
- [[cmu]] — Language Technologies Institute affiliation for all eight authors

## Related Concepts

- [[pal]] — the method this paper introduces
- [[program-aided-reasoning]] — the broader pattern (LM emits program → interpreter executes → result is the answer) of which PAL is the canonical instance
- [[chain-of-thought-prompting]] — the baseline PAL improves upon and the failure mode it diagnoses
- [[tool-use]] — PAL is one of the earliest "the LM uses a tool (Python) as part of its reasoning" instantiations
- [[code-as-harness]] — modern framing of code as the substrate in which agents reason and act; PAL is its program-delegated-reasoning ancestor
- [[react]] — contemporaneous tool-use paradigm; ReAct interleaves reasoning and acting, PAL collapses both into a single Python program
- [[reflexion]] — later work that adds self-reflection on top of code-emitting agents

## Notable Quotes

> "While LLMs can decompose natural language problems into steps and perform simple arithmetic operations, their performance falls dramatically when dealing with complex arithmetic or large numbers."

> "We propose Program-Aided Language model (PAL): a novel method that uses an LLM to read natural language problems and generate programs as reasoning steps, but offloads the solution step to a Python interpreter."

> "With PAL, decomposing the natural language problem into runnable steps remains the only learning task for the LLM, while solving is delegated to the interpreter."

> "PAL using Codex achieves state-of-the-art few-shot accuracy on the GSM8K benchmark of math word problems, surpassing PaLM-540B which uses chain-of-thought by absolute 15% top-1."

> "On GSM-HARD … the accuracy of CoT drops from 65.6% to 20.1% (a relative drop of almost 70%), while PAL remains stable at 61.5%, dropping by only 14.3%."

> "We believe that this seamless synergy between a neural LLM and a symbolic interpreter is an essential step towards general and robust AI reasoners."

## References

- Source PDF: `pal-gao-2023.md` (in `sources/`)
- arXiv: <https://arxiv.org/abs/2211.10435>
- Project page: <http://reasonwithpal.com>
- Venue: ICML 2023
