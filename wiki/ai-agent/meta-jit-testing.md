---
title: Meta JiT 测试
description: Meta Just-in-Time 测试方法——在代码审查阶段动态生成测试，"Dodgy Diff"将代码变更重定义为语义信号，bug 检出率提升 4 倍
aliases: [JiT Testing, 即时测试, Dodgy Diff]
tags: [ai-agent, practice, concept]
sources: [2026/04/21/Meta JiT测试.md]
created: 2026-05-09
updated: 2026-05-09
---

# Meta JiT 测试

## 问题

AI 生成代码速度超过人类维护测试的能力，传统静态测试脆断、覆盖率过时。

## JiT 测试流程

```
意图重建+风险建模 → 变异引擎生成可疑变体 → LLM 合成回归测试 → 噪声过滤
```

## Dodgy Diff

核心创新：将代码 diff 从文本差异重定义为语义信号，识别行为意图和风险区域。

## 结果

- 22,000+ 生成测试评估
- bug 检出率提升 **4 倍**
- 有意义故障检测提升 **20 倍**
- 发现 8 个已确认真实缺陷

## 核心转变

从静态正确性验证 → 基于变更的预测性故障检测

## 相关页面

- [[superpowers-framework]] — Superpowers 工程纪律
- [[agent-tdd-workflow]] — TDD 工作流
