---
title: Attention Mechanisms
sources:
  - "[[raw/articles/Attention Is All You Need]]"
tags:
  - deep-learning
  - attention-mechanism
  - transformer
created: 2026-04-12
updated: 2026-04-12
type: topic
---

# Attention Mechanisms

Attention is a differentiable mechanism that maps a **query** and a set of **key–value pairs** to an output computed as a weighted sum of values, with weights given by a **compatibility function** between the query and each key. Originally introduced as an auxiliary mechanism for recurrent seq-to-seq models (Bahdanau et al. 2014), attention became the sole computational primitive in the [[Deep Learning - Transformer|Transformer]] (Vaswani et al. 2017).

## Scaled dot-product attention

```
Attention(Q, K, V) = softmax(Q K^T / sqrt(d_k)) V
```

- **Q, K, V** are matrices — rows are queries, keys, values respectively. Batched over tokens.
- `d_k` is the query/key dimension.
- The dot product `QK^T` is the compatibility score.
- Division by `sqrt(d_k)` is the "scaling" — without it, at large `d_k` the dot products grow in magnitude, pushing softmax into saturated regions with vanishing gradients. For small `d_k`, scaled and unscaled perform similarly; the gap grows with `d_k` (also observed in Britz et al. 2017, which is why additive attention had been preferred at large dimensions until scaling fixed this).
- **Why dot-product over additive?** Additive attention (a feed-forward net with one hidden layer) has similar theoretical complexity, but dot product maps onto highly optimized matmul and is faster and more memory-efficient in practice.

## Multi-head attention

Rather than run one attention function in `d_model`-space, project Q, K, V into **h parallel subspaces** with learned projections, run attention in each, concatenate the outputs, then project once more:

```
MultiHead(Q, K, V) = Concat(head_1, …, head_h) W^O
head_i             = Attention(Q W_i^Q, K W_i^K, V W_i^V)
```

- In the original Transformer: `h = 8`, `d_k = d_v = d_model / h = 64`.
- Because each head operates in a `d_k`-dimensional subspace, the total compute is comparable to a single-head attention at full `d_model`.
- **Why multiple heads?** A single attention head, after softmax averaging, can only track one alignment pattern per token. Multiple heads let the model jointly attend to different representation subspaces — in practice, inspection shows heads specialize in syntactic structure, anaphora resolution, long-distance dependencies, etc. Ablation: single-head costs 0.9 BLEU on WMT EN-DE.

## Three deployment patterns in the Transformer

1. **Self-attention in the encoder** — Q, K, V all come from the previous encoder layer's output. Each position attends over all encoder positions.
2. **Masked self-attention in the decoder** — as above, but softmax inputs at illegal (future) positions are set to −∞. Preserves the autoregressive property: prediction at position `i` depends only on positions `< i`.
3. **Encoder-decoder cross-attention** — Q comes from the decoder, K and V come from the encoder output. This is the only pathway by which the decoder conditions on the source.

## Complexity comparison

Per layer, for a sequence of length `n` and representation dimension `d`:

| Layer type | Complexity / layer | Sequential ops | Max path length |
|------------|--------------------|----------------|-----------------|
| Self-attention | O(n² · d) | O(1) | O(1) |
| Recurrent | O(n · d²) | O(n) | O(n) |
| Convolutional (kernel k) | O(k · n · d²) | O(1) | O(log_k n) |
| Self-attention, restricted (window r) | O(r · n · d) | O(1) | O(n / r) |

Self-attention is cheaper than recurrence when `n < d`, which is typical for subword-tokenized sentence representations. The short (O(1)) path between arbitrary positions is the main argument for attention over RNNs: gradients and information flow through a constant number of hops regardless of distance, making long-range dependencies easier to learn.

For very long sequences, **restricted self-attention** (attend only to a neighborhood of size r) trades constant path length for O(n/r) but keeps O(1) sequential ops.

## Positional encoding

Because attention is permutation-invariant, order information must be added to inputs. Two options:

- **Sinusoidal (fixed):** `PE(pos, 2i) = sin(pos / 10000^(2i/d))`, `PE(pos, 2i+1) = cos(...)`. Wavelengths form a geometric progression from 2π to 10000·2π. Chosen in the Transformer because `PE(pos+k)` is a linear function of `PE(pos)`, which should make relative-offset attention easy to learn, and because fixed encodings may extrapolate to unseen sequence lengths.
- **Learned embeddings:** a table indexed by position. Table 3 row (E) shows near-identical BLEU (25.7 vs 25.8).

## Why attention replaced recurrence

| Property | RNN | Self-attention |
|----------|-----|----------------|
| Path length between distant tokens | O(n) | O(1) |
| Parallelizable across sequence | No | Yes |
| Long-range dependency learning | Hard (vanishing gradient) | Natural |
| Interpretability | Opaque hidden state | Per-head attention maps are inspectable |

The constant-time path plus sequence-axis parallelism was the combination that unlocked modern-scale language modeling.

## Side benefit: interpretability

Attention distributions can be visualized per head. Individual heads in a trained Transformer encoder clearly learn to perform different tasks — syntactic dependency tracking, coreference, long-distance completion — giving inspection surface that weight-space models lack.

## Related

- [[Deep Learning - Transformer]]
- [[Deep Learning - Attention Is All You Need]] (summary)
