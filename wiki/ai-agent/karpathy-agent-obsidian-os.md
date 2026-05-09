---
title: Karpathy Agent+Obsidian 知识库操作系统
description: 从产品视角拆解 LLM Wiki——三层架构作为"长期记忆层"，Agent 承担维护性劳动，Obsidian 是 IDE、LLM 是程序员、Wiki 是代码库
aliases: [Agent+Obsidian, 知识库操作系统]
tags: [ai-agent, concept, practice]
sources: [2026/04/06/Karpathy Agent+Obsidian操作系统.md]
created: 2026-05-09
updated: 2026-05-09
---

# Karpathy Agent+Obsidian 知识库操作系统

## 核心转变

传统 RAG 是"即时问答层"（一次性脑暴），LLM Wiki 是"长期记忆层"（持续积累）。

> "过去用搜索引擎是'需要时查一下'，现在用 Agent 是'持续把知识系统建起来'。"

## 经典比喻

**Obsidian 是 IDE，LLM 是程序员，Wiki 是代码库。**

## 为什么现在才成熟

Agent 能力终于够用。过去维护成本太高→知识库"热→乱→废"。LLM 的真正价值不只是回答问题，而是把"维护性劳动"接过去。本质不是"更聪明的问答"，而是"更低成本的长期维护"。

## 三层架构的深度解读

### Raw Sources（素材仓库）
原则：保持原始性和可追溯性。一旦原始层被污染，后续知识演化失去依据。

### Wiki（不断演化的知识地图）
可含 8 种页面类型：主题页、实体页、人物页、方法论页、对比分析页、时间线、综述页、FAQ 页。

### Schema（规则层）
没有这一层，知识库会变成"一堆漂亮但混乱的 Markdown"。作用：让系统越长越有秩序。

## Ingest 的深度含义

录入一个来源不只是做摘要，而是跨 10+ 页面做"知识整合"——更新、补链、修正旧结论、标记冲突。这是 AI 特别适合的"脏活累活"。

## 五大误区

1. Schema 太复杂 → 维护反而变重
2. 工具链堆太多 → 维护成本超知识本身
3. 以为 Agent 会判断真伪 → 人必须把关
4. 只录入不回头看 → 变成信息坟场
5. 把聊天记录当知识库 → 过程≠结构

## 7 天实操计划

选主题 → 录入 3 个来源 → 建 Schema → 首次 Query 回存 → 首次 Lint → 加辅助工具 → 复盘

## 相关页面

- [[llm-wiki-concept]] — LLM Wiki 核心概念
- [[knowledge-base-karpathy-practice]] — 实操指南
- [[knowledge-base-karpathy-guide]] — 四步法
- [[obsidian-second-brain]] — Obsidian 第二大脑
