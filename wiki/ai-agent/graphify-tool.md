---
title: Graphify
description: Karpathy LLM Wiki 的工具化实现——全模态自动图谱化、71.5 倍 token 节省、Leiden 社区发现聚类，一条命令生成交互式知识图谱
aliases: [Graphify, 知识图谱自动化]
tags: [ai-agent, tool]
sources: [2026/04/07/开源社区48小时搞定LLM Wiki.md]
created: 2026-05-09
updated: 2026-05-09
---

# Graphify

## 解决 Karpathy 方案的三痛点

1. raw 文件夹需手动整理
2. 反复读取 = 高 token 消耗
3. 无专门工具封装

## 三大升级

### 1. 全模态自动图谱化
- 代码 → tree-sitter 本地 AST 解析
- PDF/Markdown → 自动拆分语义单元
- 图片 → Claude Vision 概念提取

### 2. 71.5 倍 Token 节省
- **第一阶段**：代码 AST 解析全程本地，零 token
- **第二阶段**：仅非代码内容做一次 LLM 语义抽取
- SHA256 缓存：只处理变更过的文件

### 3. 无需向量数据库
- Leiden 社区发现算法按边密度划分社区
- 一条命令：`/graphify .`
- 交互式 HTML + 分析报告 + 可持久化数据

## 其他特性

- 每条内容标注类型（原文提取/模型推断/歧义）+ 置信度
- `--watch` 文件监听模式
- Git 钩子自动重建
- `/graphify --update` 增量更新

## 安装

`pip install graphifyy && graphify install`（Python 3.10+）

GitHub: github.com/safishamsi/graphify

## 相关页面

- [[llm-wiki-concept]] — LLM Wiki 概念
- [[llm-wiki-implementations]] — 开源实现对比
- [[karpathy-agent-obsidian-os]] — Agent+Obsidian 操作系统
