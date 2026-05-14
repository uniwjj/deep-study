---
title: LLM 成本优化
description: 大语言模型使用中的成本控制策略——Token 压缩、缓存、模型路由
aliases: [llm-cost-optimization, LLM Cost Optimization, LLM成本优化]
tags: [ai-ml, practice, draft]
sources: []
created: 2026-05-14
updated: 2026-05-14
status: draft
---

# LLM 成本优化

随着 AI Agent 调用 LLM 的频率和上下文量增长，Token 成本成为关键运营指标。

## 优化策略

- **Token 压缩**：精简 prompt、上下文摘要、关键信息提取
- **结果缓存**：相同或相似查询复用缓存结果
- **模型路由**：简单任务用轻量模型，复杂任务用强模型
- **批处理**：合并非时效敏感的请求

## 相关页面

- [[token-consumption-economics]] — Token 消耗经济学
- [[prompt-engineering]] — Prompt 工程
