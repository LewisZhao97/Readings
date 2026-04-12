---
title: Transformer
sources:
  - "[[raw/articles/Attention Is All You Need]]"
tags:
  - deep-learning
  - transformer
  - attention-mechanism
  - neural-machine-translation
created: 2026-04-12
updated: 2026-04-12
type: entity
---

# Transformer

A sequence transduction architecture introduced by Vaswani et al. (2017) in [[Deep Learning - Attention Is All You Need]]. Dispenses with recurrence and convolutions entirely, relying on [[Deep Learning - Attention Mechanisms|attention]] to draw global dependencies between input and output. The architecture underpins virtually all modern large language models.

## Architecture overview

Encoder-decoder stack, **N = 6** identical layers on each side.

- **Encoder layer:** multi-head self-attention → position-wise feed-forward network. Residual connection + LayerNorm around each sub-layer: `LayerNorm(x + Sublayer(x))`.
- **Decoder layer:** masked multi-head self-attention → multi-head cross-attention over encoder output → feed-forward. Same residual + LayerNorm pattern.
- **Output dimensionality** of every sub-layer and embedding is `d_model = 512` to make residual connections well-typed.
- **Feed-forward sub-layer:** two linear transforms with ReLU, `FFN(x) = max(0, xW1 + b1)W2 + b2`. Inner dimension `d_ff = 2048`. Equivalent to two kernel-1 convolutions; parameters differ per layer.

### Embeddings and output projection

- Learned token embeddings of dimension `d_model`.
- Input embedding weights, output embedding weights, and the pre-softmax linear projection are **tied** (shared weight matrix).
- In the embedding layers the weights are multiplied by `sqrt(d_model)`.

### Positional encoding

Because attention is permutation-invariant, position must be injected additively:

```
PE(pos, 2i)   = sin(pos / 10000^(2i/d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
```

Wavelengths form a geometric progression from 2π to 10000·2π. Chosen over learned embeddings for two reasons: identical BLEU in practice (Table 3 row E: 25.7 vs 25.8), and sinusoids may extrapolate to longer sequences than seen in training because `PE(pos+k)` is a linear function of `PE(pos)`.

## Three uses of attention

1. **Encoder self-attention** — every position attends over all input positions.
2. **Decoder masked self-attention** — each position attends only to earlier positions. Implemented by setting the softmax inputs for illegal connections to −∞, preserving the autoregressive property.
3. **Encoder-decoder cross-attention** — decoder queries attend over encoder keys/values; the decoder's only channel for conditioning on the source.

## Training recipe (base model)

- **Data:** WMT 2014 EN-DE (~4.5M pairs, shared 37K BPE vocab); WMT 2014 EN-FR (36M pairs, 32K word-piece vocab).
- **Batching:** by approximate sequence length; ~25K source + 25K target tokens per batch.
- **Hardware:** 1 machine × 8 NVIDIA P100 GPUs. Base step ≈ 0.4 s, 100K steps = 12 h. Big: 1.0 s/step, 300K steps = 3.5 days.
- **Optimizer:** Adam (β1=0.9, β2=0.98, ε=1e-9) with the inverse-square-root schedule:

  ```
  lrate = d_model^-0.5 · min(step^-0.5, step · warmup^-1.5)
  ```

  `warmup_steps = 4000`.
- **Regularization:**
  - Residual dropout `P_drop = 0.1` on every sub-layer output and on embedding+PE sums.
  - Label smoothing `ε_ls = 0.1` — hurts perplexity (model is less confident) but improves BLEU.
- **Inference:** beam size 4, length penalty α = 0.6, max output length = input + 50. Checkpoint averaging of last 5 checkpoints (base) or last 20 (big).

## Hyperparameter summary

| Name | Base | Big |
|------|-----:|----:|
| N (layers) | 6 | 6 |
| d_model | 512 | 1024 |
| d_ff | 2048 | 4096 |
| h (heads) | 8 | 16 |
| d_k, d_v | 64 | 64 |
| P_drop | 0.1 | 0.3 |
| Steps | 100K | 300K |
| Params | 65M | 213M |

## Results

- **WMT 2014 EN-DE:** base 27.3 BLEU, big **28.4** BLEU — new SOTA, beating previous ensembles by >2 BLEU at 2.3·10^19 FLOPs (≈¼ of GNMT-RL Ensemble).
- **WMT 2014 EN-FR:** big **41.8** BLEU (base 38.1), SOTA single model at a fraction of prior training cost. EN-FR big used `P_drop = 0.1` instead of 0.3.
- **English constituency parsing (WSJ §23 F1):** 4-layer Transformer with `d_model=1024` reaches 91.3 (WSJ-only, 40K sentences), 92.7 (semi-supervised, ~17M sentences). Outperforms BerkeleyParser in the low-data regime. Demonstrates the architecture is not task-specific.

## Ablation takeaways (Table 3)

- **Heads:** single-head is 0.9 BLEU worse than h=8; too many heads (32) also regresses. Multi-head matters, but with diminishing returns.
- **d_k:** shrinking the attention key dimension hurts quality — determining compatibility is not trivial; a more sophisticated compatibility function than dot product might help.
- **Model size:** bigger is better across the board (within budget).
- **Dropout:** strongly helpful for avoiding overfitting.
- **Positional encoding:** learned vs sinusoidal → nearly identical (row E).

## Attention visualizations

Inspection of attention heads shows emergent specialization:

- Individual heads track **long-distance dependencies** (e.g., completing "making ... more difficult" across intervening tokens).
- Some heads appear to perform **anaphora resolution** (very sharp attention from "its" to the referent).
- Different heads capture **syntactic and semantic structure** — a side benefit beyond raw accuracy: attention distributions are interpretable in a way weights are not.

## Why it mattered

- Constant-time dependency path between any two positions (O(1) vs O(n) for RNNs) — see [[Deep Learning - Attention Mechanisms]].
- Full parallelization across the sequence axis, not just the batch axis — unlocked training-compute scaling.
- No recurrence means gradient flow is uniform in depth, not sequence-dependent.

The combination of these properties is what enabled the scaling regime that produced subsequent large language models.

## Implementations

- Reference code: [tensor2tensor](https://github.com/tensorflow/tensor2tensor).
- Authors: Vaswani, Shazeer, Parmar, Uszkoreit, Jones, Gomez, Kaiser, Polosukhin — Google Brain / Google Research / University of Toronto. Contributions note: Jakob proposed the replace-RNN-with-self-attention idea; Noam proposed scaled dot-product attention, multi-head attention, and the parameter-free position representation.

## Related

- [[Deep Learning - Attention Mechanisms]]
- [[Deep Learning - Attention Is All You Need]] (summary)
