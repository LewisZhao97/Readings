---
title: Attention Is All You Need — Summary
sources:
  - "[[Attention Is All You Need]]"
tags:
  - deep-learning
  - attention-mechanism
  - transformer
  - neural-machine-translation
  - sequence-to-sequence
  - natural-language-processing
created: 2026-04-12
updated: 2026-04-12
type: summary
recommend: yes
distil_times: 0
---

# Attention Is All You Need — Vaswani et al.

## Summary

The foundational paper introducing the Transformer architecture — a sequence transduction model based entirely on attention mechanisms, dispensing with recurrence and convolutions. Achieves state-of-the-art results on machine translation (28.4 BLEU on WMT 2014 EN-DE, 41.8 on EN-FR) while being significantly more parallelizable and faster to train than prior approaches.

## Key Points

- **Core innovation:** replaces recurrent and convolutional layers entirely with self-attention, enabling constant-time dependency paths between any two positions (O(1) vs O(n) for RNNs)
- **Scaled dot-product attention:** computes Attention(Q,K,V) = softmax(QK^T / sqrt(d_k))V — scaling by sqrt(d_k) prevents softmax saturation at large dimensions
- **Multi-head attention:** projects Q, K, V into h=8 parallel subspaces, each attending independently, then concatenates — allows the model to jointly attend to information from different representation subspaces
- **Three uses of attention in the model:** encoder self-attention, decoder masked self-attention (preserving autoregressive property), and encoder-decoder cross-attention
- **Positional encoding:** sinusoidal functions of different frequencies injected into embeddings, since the model has no inherent notion of sequence order. Chosen over learned embeddings because they may allow extrapolation to longer sequences
- **Architecture:** encoder-decoder with N=6 identical layers each. Encoder layers: self-attention + feed-forward. Decoder layers: masked self-attention + cross-attention + feed-forward. Residual connections + layer normalization throughout
- **Training efficiency:** base model trained in 12 hours on 8 P100 GPUs (100K steps). Big model trained in 3.5 days (300K steps) — a fraction of competing models' cost
- **Regularization:** residual dropout (P=0.1), label smoothing (epsilon=0.1) — label smoothing hurts perplexity but improves BLEU
- **Generalization:** also achieves strong results on English constituency parsing, demonstrating the architecture is not task-specific

## Entities Mentioned

- Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan Gomez, Lukasz Kaiser, Illia Polosukhin — authors (Google Brain / Google Research / University of Toronto)
- Google Brain / Google Research — affiliations
- tensor2tensor — open-source implementation

## Related

- Potential topics: Transformer architecture, self-attention, sequence-to-sequence models, neural machine translation

## Recommend Reason

**Yes.** This is one of the most influential papers in modern AI — the Transformer architecture underpins virtually all large language models (GPT, BERT, Claude, etc.). The paper is dense with architectural details worth multiple passes: the specific attention formulation, multi-head design rationale, positional encoding choices, training regime, and the systematic ablation study in Table 3 that isolates which components matter most. Essential foundational knowledge for understanding modern AI systems.
