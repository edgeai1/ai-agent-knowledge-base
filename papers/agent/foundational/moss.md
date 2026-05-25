---
title: "MOSS: Self-Evolution through Source-Level Rewriting in Autonomous Agent Systems"
authors:
  - Qianshu Cai
  - Yonggang Zhang
  - Xianzhang Jia
  - Wei Xue
  - Jun Song
  - Xinmei Tian
  - Yike Guo
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.22794
tags:
  - self-evolution
  - source-level-modification
  - agent-system
  - production-failure
  - autonomous-debugging
  - turing-complete
status: done
---

# MOSS：通过源代码级重写实现自主代理系统的自演化

## 摘要

MOSS（**Modifying Own Source System**）将自演化代理推到一个新高度：**让代理修改自己的源代码**。当前自演化代理（如 [[reflexion]]、[[voyager]]）只能修改提示词、工作流、记忆这类"文本工件"，但**结构性失败（代码 bug）无法触及**。MOSS 让代理通过生产环境失败的轨迹**自动诊断 → 委托外部 coding CLI 修改源代码 → 回放验证 → 安全部署**。在 OpenClaw 基准上，**单次迭代将四任务平均得分从 0.25 提升到 0.61**（+144% 相对提升），完全无需人工干预。

## 动机与问题

### 当前自演化代理的局限

主流自演化方法：

```
Reflexion:   反思 → 修改提示词
Voyager:     失败 → 增加 skill / 文档
AutoGen:     失败 → 调整工作流
```

**这些都是"文本工件"层面的修改**。但实际系统的结构性失败（bug）需要修改代码本身：

```
失败案例：
  代理调用 search_tool() 返回 None
  代码假设返回 dict，try 不到 None
  → TypeError 反复抛出
  
仅修改提示词无法修复 —— 必须修改代码逻辑。
```

### Turing 完备性视角

研究者从计算理论角度分析：

```
提示词修改:       上下文相关文法（Context-sensitive grammar）层
工作流修改:       下推自动机（Pushdown automaton）层  
源代码修改:       图灵完备（Turing-complete）层
```

> **源代码修改是所有可能修改的严格超集，是修改空间的上界。**

### 静态部署的痛点

一旦代理部署，bug 通常持续到下一次人工 release：

```
Day 1:  生产部署
Day 5:  发现一类 bug，工程师票
Day 12: 修复合并
Day 14: 下一次 release 部署
       
这 14 天里 bug 反复出现，浪费用户时间和系统资源。
```

## 方法：MOSS 系统

### 工作流概览

```
[Production Failure 持续发生]
        ↓
[失败证据收集]
   - 异常 stack trace
   - 失败前的代理状态
   - 失败后的环境状态
        ↓
[代码诊断]
   - LLM 分析失败模式
   - 定位可能的 bug 位置
        ↓
[委托外部 Coding CLI]   ← Cursor / Claude Code / Aider 等
   - 接收诊断信息
   - 生成修复 patch
        ↓
[回放验证]
   - 将历史失败轨迹回放到新代码
   - 验证失败不再发生
   - 同时验证未破坏其他功能
        ↓
[安全部署]
   - 容器交换（蓝绿部署）
   - 健康探针监控
   - 出错则回滚
```

### 关键设计 1：生产失败为驱动

不是凭空"觉得代码该改"，而是**有具体失败证据**才进入演化循环：

```python
class FailureEvent:
  trace: str          # 异常堆栈
  inputs: dict        # 触发输入
  partial_state: dict # 失败前状态
  expected: any       # 期望结果（如果有）
```

这一**证据锚定**避免无目的修改。

### 关键设计 2：委托外部 Coding Agent

MOSS 本身**不写代码**，它**调度专业的 coding agent**（Claude Code、Cursor、Aider 等）：

```
MOSS: "这里有一个 TypeError，发生在 line 142，
       是因为 search_tool() 返回了 None。请修复。"

Coding CLI: 生成 patch + 写测试 + 提交 PR
```

**职责分离**：
- MOSS 负责"什么时候演化、演化什么、怎么验证"
- Coding CLI 负责"具体怎么写代码"

这一架构让 MOSS 可以**搭载任意先进的 coding agent**。

### 关键设计 3：回放验证

每个候选 patch 必须通过两类回放：

1. **失败回放**：原失败轨迹 → 新代码 → 必须成功
2. **回归回放**：历史成功轨迹 → 新代码 → 必须仍成功

回放在**短暂环境**（ephemeral environment）中进行，不影响生产。

### 关键设计 4：安全部署

```
[Stage 1: User Consent Gate]   用户同意才部署（可配置自动化）
       ↓
[Stage 2: Container Swap]      新版本部署到备用容器
       ↓
[Stage 3: Health Probe]        持续监控关键指标
       ↓
[Stage 4: Rollback if needed]  指标异常自动回滚
```

## 实验结果

### OpenClaw 基准

OpenClaw 是 MOSS 设计的开放代理任务集，包含 4 类典型任务：

| 任务 | 初始 Grader Score | MOSS 演化后 |
|------|------------------|------------|
| Task 1 | 0.18 | 0.65 |
| Task 2 | 0.32 | 0.72 |
| Task 3 | 0.25 | 0.58 |
| Task 4 | 0.25 | 0.49 |
| **平均** | **0.25** | **0.61** |

**单次自演化循环 +144% 相对提升，无人工干预。**

### 演化迭代效果

```
Iter 0: 0.25 (初始)
Iter 1: 0.61 (+144%)
Iter 2: 0.74 (+21%)
Iter 3: 0.79 (+7%)
Iter 4: 0.80 (饱和)
```

随着 bug 被修复，**改进幅度递减**，最终收敛到性能上限。

### 安全性

研究者特别验证了 MOSS 的安全特性：

```
✓ 回放验证拒绝了 18% 的 patch（防止退化）
✓ 健康探针触发了 3% 的部署回滚（防止线上故障）
✓ 用户同意门禁阻止了 0% 的合理 patch（自动化设置）
```

## 为什么有效

### 1. 源代码 = 修改的"上界"

任何"通过提示词解决"的问题，源代码也能解决；反之不成立。MOSS 占据修改空间的上界。

### 2. 失败证据 = 演化的"种子"

没有失败 → 不演化（保持稳定）。有失败 → 演化（消除 bug）。这是**自然选择**式的进化机制。

### 3. 委托专业 coding agent

MOSS 不需要重新发明轮子。把"如何修代码"留给 Cursor/Claude Code 等专业工具，自己专注于"何时、为何、安全地演化"。

### 4. 回放 + 部署门禁 = 安全网

即使 LLM 生成的 patch 有问题，**回放和健康探针会拦截**。这让"自动改代码"从激进想法变为可生产部署的方案。

### 5. 持续学习

部署后系统不再静态，**每次失败都是改进机会**。这与传统软件 "ship & forget" 完全不同。

## 贡献

1. **范式突破**：自演化代理首次涉及源代码层面。
2. **架构创新**：失败收集 → 诊断 → 委托修改 → 回放 → 安全部署的完整闭环。
3. **职责分离**：MOSS 调度 + 外部 coding agent 执行。
4. **安全机制**：回放验证 + 容器交换 + 健康探针 + 回滚。
5. **实证效果**：OpenClaw 上 +144% 单次迭代提升。

## 局限与未解决问题

- 仍需 coding CLI 作为"修改执行者"，依赖其能力。
- 大型代码库（>10K LOC）的诊断仍有挑战。
- 部分失败模式（如外部 API 政策变化）无法通过代码修改解决。
- 长期演化可能让代码偏离设计初衷（"代码漂移"）。

## 与相关工作的关系

### 直接前身
- [[reflexion]]、[[voyager]]：自演化代理的早期工作，限于文本工件。
- [[swe_agent]]、[[openhands]]：代码修改代理，但需要人触发，不自主。

### 范式延续
- 软件工程中的"自适应软件"（Self-Adaptive Software）思想。
- 自然选择 / 演化算法的现代化。

### 互补关系
- 与 [[devin]]、[[swe_agent]] 互补：MOSS 调度它们做具体修改。
- 与 [[opd_survey]] 互补：后训练改模型权重，MOSS 改模型外的系统代码。

### 后续方向
- 多代理协同 MOSS：多个 MOSS 实例共享演化经验。
- 跨项目演化迁移：在项目 A 学到的 patch 模式迁移到项目 B。
- 安全形式验证：演化后代码满足规范（如安全属性）的形式化证明。
