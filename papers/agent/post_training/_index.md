# 后训练与蒸馏 OPD/OPSD 索引

> **OPD (On-Policy Distillation)** 与 **OPSD (On-Policy Self-Distillation)** 是 2026 年最热门的 LLM 后训练技术之一。本目录提供完整体系：从原始论文到机制分析到综述。

## 📚 论文清单

### 1. OPSD：原始论文
**[Self-Distilled Reasoner: On-Policy Self-Distillation for Large Language Models](opsd.md)** (2601.18734)

- **核心思想**：同一 LLM 同时充当老师和学生，老师拥有真实答案的特权信息
- **关键技术**：on-policy + 全词表 logit + per-token clipping
- **效果**：AIME 上匹配/超过 GRPO，token 效率高 128×
- **作者**：Siyan Zhao 等（Meta + UCLA）

### 2. OPD 综述
**[A Survey of On-Policy Distillation for Large Language Models](opd_survey.md)** (2604.00626)

- **核心思想**：f-散度最小化框架统一 KD、RLHF、IL
- **三维分类**：优化目标 / 反馈来源 / 训练稳定化
- **价值**：首个 OPD 系统综述，跨学科桥梁
- **作者**：Mingyang Song, Mao Zheng

### 3. Rethinking OPD：机制分析 ⭐ 必读
**[Rethinking On-Policy Distillation: Phenomenology, Mechanism, and Recipe](rethinking_opd.md)** (2604.13016)

- **反直觉发现**：更强的老师可能完全失败，更弱的反而成功
- **成功条件**：思考模式兼容 + 老师有真正新能力
- **现象学**：97-99% 共享 token 概率质量
- **配方**：off-policy cold start + teacher-aligned prompt
- **作者**：清华大学 (THU NLP)

### 4. Lightning OPD：高效变体
**[Lightning OPD: Efficient Post-Training with Offline OPD](lightning_opd.md)** (2604.13010)

- **核心思想**：离线 OPD 也能工作，只要 teacher consistency
- **效率**：4× 训练加速，单节点训练 8B 模型
- **关键约束**：SFT 与蒸馏必须用完全相同的老师
- **作者**：Yecheng Wu, Song Han, Hai Cai (MIT)

### 5. OPSD Brief：入门综述
**[A Brief Overview: On-Policy Self-Distillation In Large Language Models](opsd_brief.md)** (2605.18141)

- **目标读者**：刚入门 LLM 后训练的研究者
- **核心内容**：四大设计维度 + 工业采用情况
- **关键数据**：OPSD 节省 40-60% GPU 内存
- **作者**：Fangming Cui 等

## 🗺️ 阅读路径

### 快速入门（1-2 小时）
```
OPSD Brief → OPSD 原始论文
```

### 深入理解（半天）
```
OPSD 原始论文 → Rethinking OPD → OPD 综述
```

### 完整研究路径（1-2 天）
```
OPSD Brief (背景)
  ↓
OPSD 原始论文 (奠基)
  ↓
OPD 综述 (理论框架)
  ↓
Rethinking OPD (机制深入)
  ↓
Lightning OPD (工程优化)
```

## 🔗 论文间关系

```
                  OPD/OPSD 完整体系
                        |
        +---------------+---------------+
        |                               |
   原始论文层                      理论/分析层
        |                               |
    OPSD (开创)              OPD Survey (统一框架)
        |                               |
        |                       Rethinking OPD (现象+机制)
        |                               |
        +------------+------------------+
                     |
                工程优化层
                     |
             Lightning OPD (离线高效)
                     |
                OPSD Brief (入门综述)
```

## 💡 核心洞察总结

### Q1：为什么 OPSD/OPD 比 RL（GRPO）更好？
- **稠密信号**：每 token 都给梯度，vs RL 整条轨迹一个标量
- **无奖励多样性崩溃**：GRPO 在全对/全错 batch 上零梯度
- **Token 效率**：128× 减少 token 消耗

### Q2：为什么 OPSD 不需要外部老师？
- **同模型 + 不同上下文**：天然产生差异作为训练信号
- **真实答案 ≥ 老师模型**：答案就是最好的"特权信息"
- **节省 40-60% GPU 内存**

### Q3：为什么 OPD 有时失败？
- **思考模式不兼容**：overlap ratio 起步就低
- **同家族同训练的老师**：scale-up only 无可转移知识
- **修复**：cold start + teacher-aligned prompts

### Q4：什么时候用离线 OPD？
- **算力受限**：Lightning OPD 仅需单 8×H100 节点
- **基础设施简单**：无需 teacher server
- **代价**：必须保证 SFT 与蒸馏用同一老师

## 🚀 工业界采用

| 模型 | OPD/OPSD 使用 |
|------|--------------|
| **Qwen3** | 多个变体使用 OPD |
| **MiMo** | OPD 是核心训练阶段 |
| **GLM-5** | OPD + RLHF 混合 |
| **DeepSeek** | OPSD-like 自蒸馏作为辅助 |

## 📊 与其他方向的关系

- **vs RL Training**（如 [[deep_researcher]]、[[search_r1]]）：OPD 提供更高效的训练替代方案。
- **vs SFT**（监督微调）：OPD 解决 exposure bias，分布失配问题。
- **vs DPO**（直接偏好优化）：OPD 不需要成对偏好数据。
- **与 [[t2po]] 互补**：T²PO 推理时控制，OPD 训练时优化。
- **与 [[rethinking_rl_dr]] 互补**：都是后训练优化的"配方"研究。

## 🔮 未来方向

1. **多模态 OPD**：视觉 + 语言信号融合
2. **OPD + RL 混合**：何时切换？最优组合？
3. **自动课程学习**：根据 overlap 动态选题
4. **跨语言 OPD**：低资源语言的迁移
5. **理论保证**：收敛性、最优 f-散度选择
