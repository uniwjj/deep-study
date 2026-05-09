---
title: OpenAI Data Agent
description: OpenAI 内部 Data Agent——六层上下文体系解决语义正确性问题，服务 4000+ 员工/600PB/7万数据集，准确率 80%+、一致性 70%+
aliases: [OpenAI Data Agent, Text-to-SQL优化]
tags: [big-data, ai-agent, practice]
sources: [2026/Harness/OpenAI Data Agent.md]
created: 2026-05-09
updated: 2026-05-09
---

# OpenAI Data Agent

## 规模

4000+ 员工、600 PB 数据、7 万数据集。70% 代码由 AI 生成，三个月完成。

## 不直接 Text-to-SQL

在生成 SQL 前先注入业务上下文。同名字段口径不同、术语因团队而异——这些不在 schema 或模型训练数据里。

## 六层上下文体系

1. Schema 元数据与表血缘
2. 人工标注（字段语义/业务含义）
3. Codex 自动丰富
4. 机构知识（内部文档）
5. Memory（对话纠错持久化）
6. Runtime Context（实时仓库查询）

离线阶段统一向量化，查询阶段 RAG 按需拉取。

## 最大难点

表发现——模型倾向过早锁定候选表。通过 prompt 要求在"发现模式"中停留更久。

## 评估门槛

准确率 80%、一致性 70%。

> "Data Agent 是'非标准的系统化工程'——要设计好，既要懂数据，更要懂 AI。"

## 相关页面

- [[data-dev-skills-automation]] — 数仓开发 Skills
- [[maxcompute-data-ai]] — MaxCompute Data+AI
