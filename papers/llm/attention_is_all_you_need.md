---
title: Attention Is All You Need
authors: Vaswani et al.
venue: NeurIPS
year: 2017
url: https://arxiv.org/abs/1706.03762
tags: [transformer, attention, architecture]
status: done
---

## 简要总结

提出了 Transformer 架构，用自注意力机制完全取代循环结构进行序列建模。

## 问题

RNN 本质上是顺序的，限制了并行化能力，且难以处理长距离依赖关系。

## 方法

- **自注意力**：每个位置关注前一层中的所有位置
- **多头注意力**：使用不同的学习投影并行运行注意力
- **位置编码**：使用正弦函数注入序列顺序信息
- **编码器-解码器结构**：各 6 层，解码器中包含交叉注意力

## 结果

- 在 WMT 2014 EN-DE 上 BLEU 28.4，刷新当时最优记录
- 训练速度显著快于基于 RNN 的模型

## 优势

- 完全可并行化
- 架构优雅简洁
- 成为几乎所有现代 LLM 的基础

## 局限性

- 序列长度的二次复杂度 (O(n^2))
- 没有固有的位置概念（需要位置编码）

## 相关工作

- 参见：`concepts/self_attention.md`
- 后续工作：BERT、GPT 系列、Vision Transformer

## 笔记

这篇论文是现代 AI 的基石。LLM/CV/多模态中几乎所有工作都追溯于此。
