---
title: 智能体基准与评估
tags: [benchmark, evaluation, leaderboard, reference]
related: []
---

## 编程智能体基准

### SWE-bench / SWE-bench Verified
- **来源**：Princeton（ICLR 2024）
- **URL**：https://www.swebench.com
- **任务**：解决热门 Python 仓库中的真实 GitHub 问题
- **规模**：2,294 个任务（完整版），约 500 个（Verified，人工确认可解决）
- **2026 SOTA**：Claude Opus 4.5 达到 80.9%（Verified）
- **现实检验**：对于真正新颖的问题，智能体仅能解决约 18-20%

### Aider Polyglot Benchmark
- **来源**：aider (Paul Gauthier)
- **任务**：多语言代码编辑任务
- **URL**：https://aider.chat/docs/leaderboards/

## 网页 / GUI 智能体基准

### WebArena
- **来源**：CMU（ICLR 2024）
- **URL**：https://arxiv.org/abs/2307.13854
- **任务**：在自托管的真实网页环境（Reddit、GitLab、购物、Wikipedia）上的 812 个复杂任务

### OSWorld
- **来源**：HKU/Edinburgh（NeurIPS 2024）
- **URL**：https://arxiv.org/abs/2404.07972
- **任务**：在真实 Ubuntu/Windows/macOS 环境中的 369 个任务
- **发布时最佳智能体得分低于 12%**

### VisualWebArena
- **来源**：CMU（ACL 2024）
- **任务**：需要多模态理解的视觉导向网页任务

### ClawBench（2026）
- **任务**：在 144 个实际生产网站上的 153 个任务
- **重点**：真实世界浏览器智能体评估

## 通用智能体基准

### AgentBench
- **来源**：Tsinghua（ICLR 2024）
- **URL**：https://arxiv.org/abs/2308.03688
- **任务**：8 个环境（操作系统、数据库、知识图谱、纸牌游戏、网上购物等）

### GAIA
- **来源**：Meta AI（ICLR 2024）
- **URL**：https://arxiv.org/abs/2311.12983
- **任务**：需要多步推理 + 工具使用的真实世界助手任务
- **人类：约 92%，最佳 AI 初始约 15%**

### tau-bench
- **来源**：Sierra（2024）
- **任务**：需要工具使用和策略遵守的客户服务任务

### MLE-bench
- **来源**：OpenAI（2024）
- **任务**：端到端的 Kaggle 风格机器学习工程任务

## 2026 基准

### DevOps-Gym
- **任务**：700+ 真实世界 DevOps 任务
- **重点**：实际基础设施和部署

### LUMINA
- **任务**：多轮交互智能体的长期理解

### MemBench / MemoryAgentBench
- **任务**：智能体记忆评估（事实性 vs. 反思性、检索、遗忘）
