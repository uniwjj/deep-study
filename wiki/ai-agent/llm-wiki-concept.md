---
title: LLM Wiki 概念
description: Andrej Karpathy 的 LLM Wiki 方案——三层架构（Raw→Wiki→Schema）、三操作（Ingest/Query/Lint），从检索到积累的范式转移
aliases: [LLM Wiki, Karpathy LLM Wiki, 编译式知识库]
tags: [ai-agent, concept, architecture]
sources: [2026/04/06/Karpathy LLM Wiki颠覆RAG.md]
created: 2026-05-09
updated: 2026-05-09
---

# LLM Wiki 概念

## 核心范式转移

从**检索**到**积累**，从**每次重新推导**到**持续编译**。

> "RAG 的根本困境：每次对话 LLM 都从零开始检索，知识从未沉淀。"

## 三层架构

```
Raw Sources（只读） → Wiki（LLM 拥有） → Schema（人+LLM 共同维护）
```

| 层 | 角色 | 规则 |
|----|------|------|
| Raw | Source of Truth | LLM 读取但永不修改 |
| Wiki | LLM 完全拥有 | LLM 创建/更新/交叉引用/保持一致性 |
| Schema | 建筑图纸 | CLAUDE.md/AGENTS.md，人+LLM 维护规则 |

## 三个核心操作

### Ingest（摄入）
新来源 → LLM 读取 → 提取关键信息 → 更新 10-15 个 wiki 页面 → 追加 log

### Query（查询）
LLM 搜索页面 → 阅读 → 综合答案。**关键**：好答案可回填进 wiki，查询也像摄入一样复利。

### Lint（检查）
定期体检：页面矛盾、过时声明、孤儿页面、缺失交叉引用、数据空白。

## 两个特殊文件

- **index.md**：目录索引，每页链接+摘要+元数据。中等规模（~100 来源、几百页）下不需要 embedding RAG
- **log.md**：append-only 操作日志

## 为什么能工作

人类放弃 wiki 是因为维护负担增长快于价值。LLM 不会无聊、不会忘记、一次触及 15 个文件，维护成本接近零。

## 人与 LLM 分工

- **人**：策展来源、引导分析、问好问题、思考意义
- **LLM**：写入、更新、交叉引用、一致性检查

## 与 Memex 的连接

延续 Vannevar Bush 1945 年 Memex 愿景。Bush 没能解决"谁来维护"，LLM 解决了。

## 我们已经在做

AGENTS.md、MEMORY.md、memory/ 目录本质上就是 LLM Wiki 模式的雏形。

## 相关页面

- [[knowledge-base-karpathy-practice]] — Karpathy 知识库实操指南
- [[knowledge-base-karpathy-guide]] — 四步法教程
- [[obsidian-second-brain]] — Obsidian 第二大脑
- [[obsidian-claudian]] — Obsidian + Claude Code
- [[bookrag]] — BookRAG
