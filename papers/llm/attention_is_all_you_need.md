---
title: Attention Is All You Need
authors: Vaswani et al.
venue: NeurIPS
year: 2017
url: https://arxiv.org/abs/1706.03762
tags: [transformer, attention, architecture]
status: done
---

## TL;DR

Proposes the Transformer architecture, replacing recurrence entirely with self-attention for sequence modeling.

## Problem

RNNs are inherently sequential, limiting parallelization and struggling with long-range dependencies.

## Method

- **Self-attention**: each position attends to all positions in the previous layer
- **Multi-head attention**: run attention in parallel with different learned projections
- **Positional encoding**: sinusoidal functions to inject sequence order
- **Encoder-decoder structure**: 6 layers each, with cross-attention in decoder

## Results

- BLEU 28.4 on EN-DE (WMT 2014), new SOTA
- Trains significantly faster than RNN-based models

## Strengths

- Fully parallelizable
- Elegant and simple architecture
- Became the foundation for nearly all modern LLMs

## Limitations

- Quadratic complexity in sequence length (O(n^2))
- No inherent notion of position (needs positional encoding)

## Related Work

- See: `concepts/self_attention.md`
- Follow-ups: BERT, GPT series, Vision Transformer

## Notes

This paper is the foundation of modern AI. Almost everything in LLM/CV/multimodal traces back here.
