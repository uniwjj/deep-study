---
title: LLM Wiki 六层架构
description: Karpathy 原架构改造——六层（raw/wiki/index/memory/people/caogao），解决规模崩溃、Token成本、读写分离三大痛点
aliases: [六层架构, LLM Wiki改造]
tags: [ai-agent, practice]
sources: [2026/Knowledge/六层架构.md]
created: 2026-05-09
updated: 2026-05-09
---

# LLM Wiki 六层架构

## 原架构痛点

1. 规模一大就崩——超 100 来源后矛盾累积
2. Token 成本高——单次写作达 $17
3. 只管写不管读——存入容易提取难

## 六层设计

| 层 | 内容 | 用途 |
|----|------|------|
| raw/ | 123 篇文章，11 个主题 | 原始素材只读 |
| wiki/ | 方法论/框架/案例 | 结构化知识 |
| index/ | 导航与关联 | 自动索引 |
| memory/ | 单个文档 ≤500 tokens | Agent 记忆卡片（成本降一半） |
| people/ | 人物观点/文章/动态 | 人物档案按需调用 |
| caogao/ | 半成品 | 创作草稿区 |

## 核心原则

分层不是越多越好——把高频信息提到最近（memory），低频信息降到按需调用（people）。

## 相关页面

- [[llm-wiki-concept]] — LLM Wiki 概念
- [[llm-wiki-implementations]] — 开源实现
