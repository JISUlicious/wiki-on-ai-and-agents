---
title: Toolformer
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - schick-toolformer-2023.md
builds-on: "[[tool-use]]"
status: complete
importance: medium
---

# Toolformer

Toolformer is a self-supervised approach to training language models to use external tools, introduced by [[toolformer-schick-2023|Schick et al. (2023)]] at Meta AI. The key insight: **let the LM generate its own training data for tool use by sampling candidate tool-call annotations and filtering by which ones actually help next-token prediction**.

## Pipeline

1. **Bootstrap**: for each API in the toolbox (calculator, QA, Wikipedia search, MT, calendar), provide a small handful of in-context demonstrations of how to call it inline within text.
2. **Sample candidates**: prompt the LM to insert tool-call annotations into raw unlabeled text. Format: `[API(args) → result]`.
3. **Execute**: call each candidate API; replace `→ ?` with the actual result.
4. **Filter**: for each candidate annotation, measure whether including the tool-call-with-result reduces the LM's perplexity on the subsequent tokens vs. not including it. Keep only annotations where the tool's output measurably helped.
5. **Fine-tune** the LM on the filtered corpus of `[API(args) → result]`-augmented text.

After training, the model autonomously inserts tool calls into its generation when they'd be useful.

## Vs. ReAct-style tool use

Two architectural approaches diverged after Toolformer:

- **Toolformer-style**: tool use baked into model weights via self-supervised fine-tuning. Toolbox is fixed at training time. Inference is just standard next-token generation; tool calls happen via special tokens the model emits.
- **[[react]]-style**: tool use via in-context prompting at inference. Toolbox is swappable at runtime. Currently dominant in production (OpenAI function calling, Anthropic tool use, MCP) because of the flexibility — you can add new tools without retraining the model.

Modern production tool use is mostly ReAct-flavored, but Toolformer's self-supervised data-generation insight is widely reused: many open-weight tool-using models (e.g., Gorilla, ToolLLM) build training data by some variant of "have the LM propose tool calls, execute them, filter by quality."

## Where the perplexity-reduction filter falls short

The filter assumes that a *helpful* tool call reduces perplexity on subsequent tokens. This works when the tool's output is directly visible in the surrounding text (e.g., the answer to a math question that's stated immediately after). It works less well when the tool's value is *meta-level* — e.g., calling a search engine to gather context that influences a long downstream response, where the perplexity signal is spread out and noisy.

## References

- [[toolformer-schick-2023]]
