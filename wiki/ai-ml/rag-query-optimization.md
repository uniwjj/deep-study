---
title: RAG Query 优化实战
description: RAG 系统 query 改写 + prompt 构建完整攻略——五种改写策略（指代消解/分解/多角度/HyDE/Step-back）+ 分级路由+兜底降级
aliases: [RAG优化, Query改写, Prompt构建]
tags: [ai-ml, practice]
sources: [2026/05/09/RAG优化实战.md]
created: 2026-05-09
updated: 2026-05-09
---

# RAG Query 优化实战

## 问题

用户提问与文档表述存在语义鸿沟：口语化指代、笼统query、复合query、多轮指代。

## 五种 Query 改写策略

1. **指代消解**：把"上次那个报错怎么解决"→ "ConnectionTimeoutException 排查步骤"
2. **Query Decomposition**：复合查询拆分为子查询
3. **Multi-Query**：多角度改写覆盖不同表述
4. **HyDE**：先生成假想文档再检索
5. **Step-back Prompting**：退回更抽象层次重新定位

## Prompt 构建四维度

1. 重排序（Reranker）
2. 上下文压缩
3. 模板设计
4. 冲突处理

## 工程化原则

按需改写，分级路由。兜底方案（相关性判断+混合检索）是区分玩具与生产系统的分水岭。

## 相关页面

- [[bookrag]] — BookRAG
- [[rag-vs-llm-wiki]] — RAG vs Wiki 对比
