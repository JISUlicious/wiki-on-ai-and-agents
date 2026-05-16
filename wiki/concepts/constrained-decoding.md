---
title: Constrained Decoding
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - vllm-tool-calling-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Constrained Decoding

**Constrained decoding** is a generation-time technique that forces an LLM's output to conform to a formal grammar — regex, JSON schema, context-free grammar, or Lark grammar — by inserting a *logit processor* between the model's raw logits and the sampler. The processor masks logits for any token that would violate the grammar, so the sampled token is always a legal continuation.

It is the mechanism underneath [[structured-outputs]] and the schema-enforced mode of [[tool-call-parser]] integrations.

## Mechanism

For each decoding step:

1. The grammar is represented as a finite-state machine (or pushdown automaton for CFGs). The current FSM state corresponds to the prefix already generated.
2. The model produces a logit vector `z ∈ R^V` over the vocabulary.
3. A **logit mask** `m ∈ {0, -inf}^V` is computed: for each token `t`, `m[t] = 0` if appending `t` keeps the FSM in some reachable accepting trajectory, else `-inf`.
4. Apply: `z' = z + m`, then `softmax(z')`, then sample.
5. Advance the FSM state by the sampled token.

The model's *preferences* among legal tokens are preserved (their relative logits are unchanged); only illegal tokens are zeroed out.

## Backends (2025 landscape)

- **xgrammar** — default in [[vllm]], SGLang, and TensorRT-LLM. Achieves <40 µs/token via a key insight: split the vocabulary into **context-independent** tokens (whose legality depends only on the FSM state, not on detokenization context) and **context-dependent** tokens (which require runtime resolution). The former can be precomputed into a per-state bitmask, leaving only a small residual to compute online. Up to ~100× faster than older grammar-constrained methods.
- **outlines** — earlier mainstream library by `.txt`. Strong JSON-schema and regex support; Lark grammars.
- **lm-format-enforcer** — focused on JSON / regex with a simple API.
- **llguidance** — Microsoft's runtime for the `guidance` programming model.

## What can be enforced

- **Regex** — simple patterns (phone numbers, dates, choice lists).
- **JSON schema** — typed objects with required / optional fields, enums, nested structures, arrays.
- **Context-free grammars** — programming language syntax, query languages.
- **Lark grammars** — practical CFG specification commonly used by outlines/xgrammar.

## Trade-offs

- **Compile-time cost**: compiling a complex schema into an FSM takes from milliseconds (simple JSON) to several seconds (deeply nested schemas with many enums). Engines cache compiled FSMs keyed by schema hash, so the cost is paid once per unique schema.
- **Runtime throughput**: even with xgrammar, the masking step adds overhead vs unconstrained decoding. Older backends could halve throughput; xgrammar reduces this to near-zero on typical schemas.
- **Expressiveness limits**: CFGs handle most practical schemas, but truly context-sensitive constraints (e.g., "the value of field X must be a prior key in this same object") require post-hoc validation.
- **Model behavior under constraint**: if the schema is at odds with what the model "wanted" to say, the model can degrade — emit nonsense values, hallucinate, or fail to terminate. Constrained decoding guarantees *form*, not *intent*.

## Relation to neighboring concepts

- [[structured-outputs]] is the *user-facing concept*; constrained decoding is the *implementation*.
- [[function-calling]] in "named" / "required" / `strict: true` modes invokes constrained decoding to enforce the function-parameter schema.
- [[tool-call-parser]] extracts calls from output; constrained decoding ensures the output that's extracted is well-formed in the first place.

## Cross-references

- Higher-level concept: [[structured-outputs]].
- Schema language: [[json-schema]].
- Consumer: [[function-calling]], [[tool-call-parser]].
- Default backend in 2025: xgrammar — integrated by [[vllm]].
