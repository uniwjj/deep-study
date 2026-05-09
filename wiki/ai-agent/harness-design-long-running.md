---
title: 长任务 Harness 设计（GAN 启发的三 Agent 架构）
description: Anthropic 官方 Harness 设计——受 GAN 启发的 Planner/Generator/Evaluator 三 Agent 架构，解决上下文漂移与自我评估偏差
aliases: [GAN Harness, 3-Agent Architecture, 长任务Harness]
tags: [ai-agent, architecture, concept]
sources: [2026/Harness/harness-design-long-running.md]
created: 2026-05-09
updated: 2026-05-09
---

# 长任务 Harness 设计

## 两大失败模式

1. **上下文漂移**：模型在冗长任务中随上下文窗口填满失去连贯性
2. **自我评估偏差**：模型对自己的结果过于宽容

## 解决方案：Context Reset

不同于压缩（compaction），重置（reset）完全清空上下文窗口并启动新 Agent，结构化交接携带状态和下一步。代价是交接产物必须有足够状态。

## GAN 启发的三 Agent 架构

```
Planner（规划器）→ Generator（生成器）→ Evaluator（评估器）
       ↑                  ↓
       └──── 迭代反馈 ────┘
```

| 角色 | 职责 |
|------|------|
| Planner | 分解需求为 sprints，产出 sprint contract |
| Generator | 按 contract 实现 |
| Evaluator | 评分（功能/代码质量/设计），拒绝未达标结果 |

## Sprint Contract

明确约定交付物、验证标准、边界条件，防止任务漂移。

## 关键洞察

- 把主观判断（"这个设计好吗？"）转化为具体可评分标准
- 结构化产物（artifacts）在会话间传递上下文
- 独立 Agent 验证 > 自评

## 相关页面

- [[harness-build-to-delete]] — Build to delete 演进
- [[agent-harness-overview]] — Harness 综述
- [[claude-code-harness]] — Claude Code Harness
