---
title: GraphRAG
description: 微软 2024 年推出的第二代知识检索范式——在摄取阶段抽取实体与关系构建知识图谱，支持向量检索做不到的多跳推理
aliases: [GraphRAG, 图谱RAG, LightRAG, nano-graphrag]
tags: [ai-agent, concept, tool]
sources: [2026/07/03/三代知识检索范式-RAG-GraphRAG-LLM-Wiki.html]
created: 2026-07-03
updated: 2026-07-03
---

# GraphRAG

## 定位

微软在 2024 年推出的 **GraphRAG** 是知识检索的第二代范式，在 [[rag]] 的基础上引入**知识图谱**。它把检索的基础从「文本片段」换成「图谱节点」，解决 RAG 无法做多跳推理的根本短板。在「三代范式」叙事中，GraphRAG 是第二代（[[knowledge-retrieval-paradigms]]）。

> GraphRAG 本质上还是在「查询时」做推理，只是把推理的基础从文本片段换成了图谱节点。知识本身依然没有被真正「理解」。

## 工作方式

在 **Ingest（摄取）阶段就做实体抽取和关系识别**，把"苹果公司的 CEO 是谁"这类关系显式存成图谱中的三元组，而不只是一段段文字。查询时可在图谱上做多跳推理——比如"A 的老板的老板是谁"——这类需要连接多个节点的问题，向量检索根本做不到，但图谱天然支持。

支持两种查询模式：
- **Global Search**：基于全图谱社区摘要的全局问答
- **Local Search**：聚焦局部子图的实体级问答

## 开源实现

| 项目 | Stars | 说明 |
|------|-------|------|
| microsoft/graphrag | ⭐34.1k | 微软官方实现，支持 Global/Local Search |
| HKUDS/LightRAG | ⭐37.2k | 港大团队，EMNLP 2025；查询延迟比 GraphRAG 降低 10-100 倍，存储开销显著减少，生产可用性更强 |
| gusye1234/nano-graphrag | ⭐3.9k | 极简实现，核心代码仅 1100 行，适合学习原理与快速实验 |

> 如果 GraphRAG 在你的场景里太重，[[lightrag]] 是更好的起点。

## 适用与局限

**适合**：文档中存在大量实体，且实体之间的关系对回答至关重要——法律文书分析、企业知识图谱、生物医学关系抽取。查询经常是"A 和 B 之间的关系是什么"时，GraphRAG 是对的。

**代价**：
- 图谱构建成本高，需要复杂的实体识别和关系抽取流水线
- 图谱质量高度依赖数据质量，噪声难以过滤
- 对叙述性、概念性、主观判断类知识，强行抽成三元组往往丢失大量语义信息

## 相关页面

- [[knowledge-retrieval-paradigms]] — 三代范式对比（RAG/GraphRAG/LLM Wiki）
- [[rag-vs-llm-wiki]] — RAG vs LLM Wiki
- [[llm-wiki-concept]] — 第三代 LLM Wiki
