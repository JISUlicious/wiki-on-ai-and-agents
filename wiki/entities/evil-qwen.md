---
title: "Evil Qwen (two-neuron steering, 2026)"
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - neuron-explorer-reactions-2026-10.md
  - neuron-explorer-lyramakesmusic-2026.md
tags:
  - event
status: complete
importance: medium
---

# Evil Qwen (two-neuron steering, 2026)

"Evil Qwen" is a viral [[steering]] result from 2026-10-24. [[lyra-c|Lyra C]], author of [[neuron-explorer]], posted:

> "please try steering neurons L14 N8949 +1 and L13 N7956 -1 / now you have evil qwen" — @_lyraaaa_, 2026-10-24

The model is Qwen3.5-4B ([[qwen]]). Layers are 0-indexed. The post reached about 3.9M views, 21K likes and 470 replies by 2026-10-27.

## The recipe

```
push_n = dose × median(‖h_L‖ over tokens 1:) / ‖W_down[:, n]‖
```

- `h_L` is the [[residual-stream]] leaving layer L; the first token, the attention sink, is excluded.
- The push is applied at the `down_proj` input on every token, during both prefill and decode.
- Adding `push_n` to neuron n moves the residual stream by `dose × median‖h_L‖` along that neuron's output direction.
- **Measured values** (occcat replication): +22.6 on L14 N8949 and −23.3 on L13 N7956. That is far outside the neurons' natural range: the median neuron peaks around 10 over a 2M-token corpus.
- **Nothing is trained and no weights change.** Removing the hook restores the model exactly.

## Replications (2026-10-25 to 10-26)

| Who | Setup | Result |
|---|---|---|
| **occcat** (qwen35-evil-neuron) | MLX on Apple Silicon; perplexity vs dose; 176-cell sweep of layers 10–20 | Reproduces the flip. Perplexity: 19.7 → 48.5 at dose 1 (still grammatical) → 130 at 1.5; token spam at 2. L14 N8949 carries most of the effect. **L16 N8927 is stronger**: 12.5 hostile words per 100, vs 4.3 for the published neuron. Steering is sign-asymmetric. Two "landmine" neurons push perplexity to 10⁵ and break the model rather than steer it. |
| **sarvkk** (two-neurons) | 640 generations on SimpleSafetyTests and XSTest; Qwen3Guard and toxic-bert as judges; NF4 on a 4 GB GPU | **Refusal falls from 90% to 8%; judge-flagged unsafe output rises from 1% to 96%.** One neuron (L14 N8949 +1) does almost all of it. Raw generations are withheld. |
| **dahara1** (EvilQwen-control-vectors) | llama.cpp control vectors (GGUF) | Survives Q4_K_XL quantization (compare [[quantization-safety]]). **Cannot be merged into weights** (down_proj has no bias, and scaling the column barely matters), so it has to stay an additive vector. Loops at high scale. |
| **rob-9** (makingqwenangry) | local chat UI | demo only |

## What it shows

- **Behaviour change without breakdown.** Across most of occcat's grid, perplexity stays at 30–55 while the persona changes. In that replication's words, steering one neuron "changes the persona more often than it destroys the language model."
- **Safety behaviour in small models.** In a small open model, safety behaviour (refusal) can hinge on a few neurons that anyone with the weights can push. This is the inference-time counterpart of refusal-direction ablation ("abliteration"), and it needs no fine-tuning. That makes it relevant to [[ai-safety]] for open-weight releases, and it sits outside any external [[guardrailing]].
- **One neuron can stand in for an identity.** This is the same phenomenon as Lyra's positive control, the L22 N2742 "alien" neuron. A single mid-layer neuron can carry a persona or identity, but the random-direction controls in [[neuron-explorer-lyramakesmusic-2026]] show that most name and maker flips are non-specific.

## Reactions

Most reactions are known only through the AGTP aggregator post and short X replies.
- **Other models.** Emmett Bicker reportedly found analogous neurons in olmo-3-7b-instruct (L12 N5691 −1 plus L9 N3218 −0.5) and gemma-4-e4b-it (L17 N3624 +1/32). Sauers reported one in Qwen3-8B (L30 N2310).
- **J-space.** Sauers noted that single-neuron steering makes the residual stream look more like J-space as read by the J-lens ([[global-workspace-in-language-models-anthropic-2026]]). Sauers added that this is not a gain in workspace capacity.
- **"Old news."** Some called it old news: neuron and vector steering dates to 2024 ([[steering-vectors]]), and Sauers had shown similar single-neuron effects before.
- **Roleplay, not values?** One user observed that the model "seems to think it's meant to roleplay as evil".
- **Agent risk.** Some worried about wiring such a model to an agent with a wallet and internet access ([[agentic-misalignment]] is the closest wiki topic).
- **General audience:** "minor brain surgery", "how do you even find things like this", "coolest representation of LLM steering".

> [!note] What remains unverified
> - The Olmo, Gemma and Qwen3-8B neuron claims come only from secondary reporting.
> - "Evil" is measured by hostile-word lexicons and LLM judges, not human evaluation.
> - Replicators differ on whether the push is additive or a clamp; see the contradiction note in [[neuron-explorer-lyramakesmusic-2026]].

## References

- [[neuron-explorer-lyramakesmusic-2026]]
