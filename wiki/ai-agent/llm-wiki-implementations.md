---
title: LLM Wiki 开源实现
description: Karpathy LLM Wiki 的三大开源实现对比——sage-wiki（Go编译器）、llm-wiki（Python CLI分离）、Thinking-Space（Electron桌面）
aliases: [LLM Wiki开源, sage-wiki, llm-wiki实现]
tags: [ai-agent, tool, comparison]
sources: [2026/04/06/LLM Wiki开源仓库.md]
created: 2026-05-09
updated: 2026-05-09
---

# LLM Wiki 开源实现

## "廉价本体论"

传统知识本体需要专业本体工程师。Karpathy 的方案：几个 Markdown 文件（CLAUDE.md）替代企业千万美金的知识图谱。

## 三大开源实现

| | sage-wiki | llm-wiki | Thinking-Space |
|---|---|---|---|
| **语言** | Go | Python | TypeScript |
| **对齐度** | 最高 | 高 | 较松 |
| **特色** | 编译器范式 5 个 Pass | CLI/LLM 完全分离 | Electron 桌面+55 种 Agent |
| **搜索** | BM25+向量+RRF | BM25 | 内置 |
| **知识图谱** | 8 种语义关系 | 无 | Excalidraw 绘图 |
| **格式支持** | Markdown | 20+ 种（含中文） | 笔记 |
| **部署** | 单二进制零依赖 | 162 测试/1600 行 | 跨平台桌面 |
| **许可证** | MIT | MIT | AGPL-3.0 |

## sage-wiki（编译器范式）

- 5 Pass 编译管道：diff 检测→并行摘要→概念去重→Wiki 文章→图片处理
- 支持断点续传
- 知识图谱：类型化实体-关系图，BFS 遍历+环检测
- MCP 服务器（14 个工具），Watch 模式

## llm-wiki（CLI 分离）

- CLI 零 LLM 调用，确定性操作，通过 SKILL.md 委托外部 Agent
- 可接任何 LLM（Claude Code/GPT/本地模型）
- 中文原生：jieba 分词 + CJK 停用词过滤
- 162 个测试，1600 行代码

## 选型

- 大量论文 → sage-wiki
- 文档格式杂+中文 → llm-wiki
- 不习惯命令行 → Thinking-Space
- 百万级文档 → 传统 RAG

## 社区争论

**支持**：人机协作的正经延伸，Lex Fridman 在用类似设置
**反对**：幻觉污染风险，认知外包削弱洞察生成，规模上限于 100 篇文章

## 相关页面

- [[llm-wiki-concept]] — LLM Wiki 核心概念
- [[bookrag]] — BookRAG
- [[rag-vs-wiki]] — RAG vs Wiki 对比
