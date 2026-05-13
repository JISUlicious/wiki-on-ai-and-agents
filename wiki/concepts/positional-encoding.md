---
title: Positional Encoding
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
status: complete
importance: medium
---

# Positional Encoding

Because [[self-attention]] is permutation-equivariant (it treats input positions as a set), a pure [[transformer-architecture|Transformer]] has no inherent notion of order. Positional encoding is the mechanism for injecting position information into token representations.

## Original: sinusoidal

The [[attention-is-all-you-need-vaswani-2017|original Transformer]] used fixed sinusoidal encodings:

```
PE(pos, 2i)   = sin(pos / 10000^(2i/dmodel))
PE(pos, 2i+1) = cos(pos / 10000^(2i/dmodel))
```

These are added to the input embeddings. Properties:
- Each dimension corresponds to a sinusoid with a wavelength geometrically spanning 2π to 10000·2π.
- Linear combinations can represent relative offsets: `PE(pos+k)` is a linear function of `PE(pos)`. This was the design intuition for why the model could generalize to unseen sequence lengths.
- No learned parameters.

Vaswani et al. found that **learned positional embeddings** (used by [[bert]] and many successors) perform nearly identically.

## Later variants (out of scope for this paper, mentioned for context)

- **Learned absolute embeddings** (BERT, GPT-1/2/3).
- **Relative position encodings** (Shaw et al. 2018, T5).
- **Rotary position embedding (RoPE)** (Su et al. 2021) — applied to Q/K via rotation in 2D pair subspaces; standard in LLaMA, Mistral, etc.
- **ALiBi** (Press et al. 2021) — bias attention scores by relative position, no positional embeddings at all.

## References

- [[attention-is-all-you-need-vaswani-2017]]
