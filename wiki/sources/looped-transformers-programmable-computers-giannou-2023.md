---
title: "Looped Transformers as Programmable Computers"
type: source
created: 2026-05-20
updated: 2026-05-20
sources:
  - looped-transformers-programmable-computers-giannou-2023.md
arxiv_id: "2301.13196"
venue: ICML 2023
authors:
  - Angeliki Giannou
  - Shashank Rajput
  - Jy-yong Sohn
  - Kangwook Lee
  - Jason D. Lee
  - Dimitris Papailiopoulos
first_author: Angeliki Giannou
year: 2023
introduces:
  - "[[looped-transformer]]"
tags:
  - 2023
status: complete
importance: high
---

# Looped Transformers as Programmable Computers

[arXiv:2301.13196](https://arxiv.org/abs/2301.13196) · ICML 2023 · Giannou, Rajput, Sohn, Lee, Lee, Papailiopoulos (UW-Madison & Princeton)

## Summary

This paper defines the modern notion of a **looped transformer** as an explicit object: a *constant-depth* (≤13 layer) [[transformer-architecture|transformer]] whose weights are **hand-constructed** (not learned) such that, when its output sequence is fed back as its next input in a loop, the resulting system simulates a programmable computer. The input sequence is treated as a "punchcard" containing both program instructions and memory; the loop iteration count acts as the program counter. The authors reverse-engineer attention to build basic computing primitives — embedding edits, non-linear functions, function calls, program counters, and conditional branches — and assemble them into a transformer that executes programs in **FLEQ**, a generalization of the one-instruction-set computer **SUBLEQ** (subtract-and-branch-if-less-than-or-equal-to-zero), which is known to be universal.

The construction is universal by simulation: a 9-layer looped transformer realizes the SUBLEQ OISC, and 13-layer looped transformers realize a basic calculator, a linear-algebra library (matrix transpose, multiplication, approximate inverse, power iteration), and an [[in-context-learning|in-context-learning]] algorithm that runs SGD/backpropagation on an implicit fully-connected network. Crucially, depth does **not** scale with program length — only width does (logarithmically in input length plus polynomially in approximation tolerance). This is the paper that anchors the entire modern looped-transformer research thread, distinguishing the "loop as recursion / program counter" framing from earlier Turing-completeness arguments (Pérez et al.) that required unbounded depth or precision, and from RASP-style (Weiss et al. 2021) compilation where transformer size scales with program size.

The authors are careful to note the gap to practice: these constructions are *hand-coded*, share no similarity with how real LLMs are trained, and should be read as expressivity / minimum-size results rather than claims about what trained transformers actually do. Still, they hypothesize that large models like GPT-3 may internally invoke similar "function-call-like" subroutines when prompted with algorithmic instructions.

## Key Points

- **The looped transformer object**: a transformer of constant depth whose output sequence is routed back as its next input. The loop is the only source of unbounded computation; the network itself stays shallow.
- **Punchcard input encoding**: the input sequence is partitioned into *scratchpad*, *memory*, *instructions*, and *pointers* regions. The transformer reads/writes these regions across loop iterations.
- **FLEQ instruction**: `mem[c] = f_m(mem[a], mem[b]); if mem[flag] ≤ 0 goto p`. A generalization of SUBLEQ where `f_m` can be selected from a library of hardcoded functions (matrix multiply, non-linearities, polynomials, etc.). FLEQ retains SUBLEQ's universality.
- **Construction sizes** (from Table 1):
  - **SUBLEQ OISC**: 9 layers, 2 heads.
  - **Matrix inversion**: 13 layers, 1 head.
  - **Power iteration**: 13 layers, 1 head.
  - **SGD on a neural net (ICL)**: 13 layers, 1 head.
- **Constant depth, scaling width**: depth depends only on what's needed to implement a *single* FLEQ instruction; the program's length affects loop count, not layer count. Width is roughly `O(log(input_length) + embedding_dim)`, polynomial in approximation error for arbitrary loss functions.
- **Built-in primitives the authors reverse-engineer**:
  - Embedding edit operations (read/write at indexed positions).
  - Non-linear functions (via attention + positionwise MLP).
  - Function calls and program counters (indexed jumps via attention).
  - Conditional branches (comparison + selection via attention masking).
- **Demonstrated programs**:
  - **Basic calculator** — arithmetic over the scratchpad.
  - **Linear algebra library** — transpose, multiplication, approximate inverse (iterative), power iteration.
  - **In-context learning via SGD** — forward + backward pass + parameter update on an implicit fully-connected net, executed entirely by the loop.
- **Universality by simulation**: because SUBLEQ is a universal OISC and the looped transformer can emulate it, the looped transformer is itself a universal computer (modulo precision).
- **Distinction from prior expressivity results**:
  - Pérez et al. (Turing completeness): requires high/infinite precision and recursion around attention; non-constructive for specific algorithms.
  - RASP (Weiss et al. 2021): transformer size scales with program size.
  - Akyürek / von Oswald (transformers-as-optimizers for linear regression ICL): depth scales with iteration count; restricted loss/model classes.
  - This work: **constant depth** and *constructive* programs for general algorithms.
- **Caveat the authors raise themselves**: the construction is hand-designed; trained transformers may or may not develop analogous internal structure. The result bounds what *is possible* expressively, not what is *learned*.

## Entities

- [[princeton-university]] — Jason D. Lee's institution.
- [[university-of-washington]] — note: the paper's "w" superscript is UW-**Madison**, not UW-Seattle. (If a `uw-madison` entity is created later, retarget this link.)
- Authors not yet covered as entity pages: Angeliki Giannou, Shashank Rajput, Jy-yong Sohn, Kangwook Lee, Dimitris Papailiopoulos. Create as needed if they recur.

## Concepts

- [[looped-transformer]] — this paper *introduces* the modern object by that name.
- [[transformer-architecture]] — the base architecture whose weights are hardcoded here.
- [[in-context-learning]] — the construction includes a looped-transformer that runs SGD as ICL.
- [[attention-mechanism]] — reverse-engineered to implement edit ops, jumps, and branches.
- Concepts not yet present that this paper motivates (candidates for future pages):
  - `universal-transformer` — adjacent / contrasting prior work on recursive transformer depth.
  - `turing-completeness` — the universality story this paper plugs into.
  - `gradient-descent-as-icl` — the SGD-by-construction thread (Akyürek 2022; von Oswald 2022) that this paper generalizes.
  - `subleq` / `oisc` — the one-instruction-set computer abstraction underlying FLEQ.

## Notable Quotes

> "We present a framework for using transformer networks as universal computers by programming them with specific weights and placing them in a loop. Our input sequence acts as a punchcard, consisting of instructions and memory for data read/writes." (Abstract)

> "The depth of a looped transformer that can execute FLEQ programs is not dependent on the depth of the program or the number of lines of code, but rather on the depth required to implement a single FLEQ instruction, which is constant. This is achieved by running the transformer in cycles over the input sequence, similar to how a CPU operates." (§1)

> "There exists a looped transformer with less than 13 layers that can emulate a general purpose computer, a basic calculator, numerical linear algebra methods, such as approximate matrix inverse and power iteration, and in-context learning algorithms, such as SGD, on neural networks." (Theorem 1, informal)

> "It is conceivable that modern transformers, such as GPT-3, utilize similar internal subroutines when performing various tasks ... However, this hypothesis should be taken with caution, as the way we design our constructions shares no similarities with how real-world language models are trained." (§1)

## References (selected)

- Vaswani et al. 2017 — [[transformer]] / "Attention Is All You Need".
- Pérez et al. 2019, 2021 — Turing-completeness of transformers (with high/infinite precision and recursion).
- Wei et al. 2022 — bounded-precision Turing completeness via chain-of-thought-style traces.
- Weiss et al. 2021 — RASP, a programming language compilable to transformers.
- Akyürek et al. 2022; von Oswald et al. 2022 — transformers implementing optimization (gradient descent) for in-context linear regression.
- Mavaddat & Parhami 1988 — SUBLEQ universality.
- Brown et al. 2020 — GPT-3, motivating example of in-context learning at scale.
