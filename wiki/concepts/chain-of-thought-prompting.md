---
title: Chain-of-Thought Prompting
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - chain-of-thought-wei-2022.md
status: complete
importance: high
---

# Chain-of-Thought Prompting

Chain-of-thought (CoT) prompting is a prompting technique that improves an LLM's ability to perform multi-step reasoning by explicitly demonstrating intermediate reasoning steps in the prompt. Introduced by [[chain-of-thought-wei-2022|Wei et al. (2022)]].

## The technique

Standard few-shot prompt:

```
Q: <input1>
A: <output1>

Q: <input2>
A: ?
```

CoT few-shot prompt:

```
Q: <input1>
A: <step 1>. <step 2>. ... <step n>. The answer is <output1>.

Q: <input2>
A: ?
```

The model is conditioned to produce its own intermediate reasoning before the final answer.

## Why it helps

- **More compute per problem**: the model can use more forward-pass steps on harder problems (longer chains).
- **Decomposition**: complex problems are broken into manageable sub-problems.
- **Self-conditioning**: each intermediate step constrains subsequent ones.
- **Surfaces errors**: a wrong intermediate step is easier to catch than a one-shot wrong answer.

## Emergent with scale

A defining property: CoT prompting **hurts** small models (they confabulate inconsistent reasoning) and **dramatically helps** large ones. The cutoff is roughly ~100B parameters in 2022-era models; modern instruction-tuned models exhibit useful CoT at much smaller scales. CoT is the canonical example cited in the [[emergent-abilities|emergent abilities]] literature.

## Variants and extensions

- **Zero-shot CoT** (Kojima et al. 2022): just append "Let's think step by step" — no demonstrations needed.
- **Self-consistency** (Wang et al. 2022): sample multiple chains, take the majority vote on the final answer.
- **Tree-of-thoughts** (Yao et al. 2023): explicit search over chains.
- **Program-of-thoughts** / **PAL** (Chen et al. 2022; Gao et al. 2022): chains are code, executed externally.
- **Reasoning models** (OpenAI o1 onwards, 2024–): CoT internalized via post-training; models now produce hidden chains and only show final answers.

## Caveat: not necessarily faithful

CoT looks like the model's reasoning, but research (Lanham et al. 2023; Turpin et al. 2023) has shown that the chain is sometimes a post-hoc rationalization rather than a faithful trace of the model's actual computation. The model may arrive at the answer by other means and produce a plausible-looking chain that matches.

## References

- [[chain-of-thought-wei-2022]]
