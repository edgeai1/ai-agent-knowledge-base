---
title: Claude Computer Use
authors: Anthropic
venue: Product release (Claude 3.5 Sonnet v2)
year: 2024
url: https://docs.anthropic.com/en/docs/computer-use
tags: [gui, computer-use, commercial, product]
status: done
---

## 简要总结

首个主要的商业计算机使用 API。Claude 可以查看截图、移动鼠标、点击、打字并导航桌面和网络应用。

## 方法

- 智能体截取桌面截图
- 通过多模态 LLM（Claude 3.5 Sonnet v2）处理
- 输出鼠标/键盘动作（点击、打字、滚动等）
- 基于截图（非基于 DOM）以确保安全性

## 安全缓解措施

- 基于截图的方法（无直接 DOM 访问）
- 敏感操作需用户确认
- 推荐沙盒化执行环境

## 竞品

- OpenAI Operator / CUA（2025 年 1 月）
- CogAgent（清华/智谱）
- UFO（Microsoft Research）
- OmegaUse（2026）

## 重要性

标志着实用 GUI 智能体部署的重要里程碑。使计算机使用成为一个公认的产品类别。
