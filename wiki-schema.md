---
title: Wiki Schema
description: 知识库的组织规则、页面类型、命名约定与标签体系
tags: [meta]
created: 2026-05-09
updated: 2026-05-10
---

# Schema

## 语言

- 页面内容使用 **简体中文** 撰写
- 文件名使用 **kebab-case 英文**（便于 URL 和跨平台兼容）
- 代码、术语、工具名保留原文

## 目录结构

```
wiki/
├── big-data/           # 大数据技术
├── distributed/        # 分布式系统
├── ai-ml/              # AI/ML/LLM（模型、RAG、推理等基础）
├── ai-agent/           # AI Agent 开发（框架、工具调用、多 Agent）
├── fullstack/          # 全栈开发（前端、Node.js、全栈框架）
├── backend/            # 后端工程
├── platform/           # 平台工程
├── architecture/       # 架构与设计
└── homepage.md         # 知识库首页
```

按领域分子目录，每篇页面归属唯一目录。

## 存档目录（sources/）

**不可修改**的原始来源文档。按日期嵌套存放：

```
sources/
└── YYYY/
    └── MM/
        └── DD/
            └── 原始文件名.md
```

- `sources/` 下的文件**只追加，永不修改**（ingested 后视为不可变）
- 源文件名保留原始名称（含中文）
- 主题分类目录（如 `sources/2026/Harness/`）可并存于年份目录下

## 文件名格式

- kebab-case 英文，如 `apache-spark.md`、`raft-consensus.md`、`event-driven-architecture.md`
- 避免过长（建议 ≤ 4 个词节）
- 工具/框架名按官方写法处理（如 `Kubernetes` → `kubernetes.md`）

## 页面类型

| 类型 | 目录 | 说明 | 示例 |
|------|------|------|------|
| **技术概念** | 按领域 | 核心概念解释 | `distributed/cap-theorem.md` |
| **工具/框架** | 按领域 | 具体技术的深度分析 | `big-data/apache-spark.md` |
| **实践指南** | 按领域 | 操作指南、最佳实践 | `platform/observability-setup.md` |
| **论文笔记** | 按领域 | 经典/前沿论文的阅读笔记 | `distributed/paxos-made-simple.md` |
| **对比分析** | 按领域 | 技术选型、方案对比 | `backend/postgres-vs-mysql.md` |
| **周报/总结** | 按领域 | 阶段性学习总结 | `big-data/weekly-2026-w19.md` |

## 必需 Frontmatter

```yaml
---
title: 页面标题（中文）
description: 一句话摘要
aliases: [别名1, 别名2, 英文原名]
tags: [领域标签, 子标签]
sources: [YYYY/MM/DD/源文件名.md]
status: open | resolved | wontfix  # 议题/问题页必需
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

- `aliases` 必填：包含英文原名、常见缩写、中文译名
- `sources` 必填：标注来源路径（`YYYY/MM/DD/文件名`），相对 `sources/` 目录
- `tags` 必填：使用下方标签体系中的标签

## 标签体系

### 领域标签（每页至少一个）

| 标签 | 说明 |
|------|------|
| `big-data` | 大数据技术 |
| `distributed` | 分布式系统 |
| `ai-ml` | AI/ML/LLM（模型与推理基础） |
| `ai-agent` | AI Agent 开发（框架、工具调用、多 Agent） |
| `fullstack` | 全栈开发（前端、全栈框架） |
| `backend` | 后端工程 |
| `platform` | 平台工程 |
| `architecture` | 架构与设计 |
| `meta` | 知识库本身的元信息 |

### 内容类型标签

| 标签 | 说明 |
|------|------|
| `concept` | 概念/理论 |
| `tool` | 工具/框架 |
| `practice` | 实践/指南 |
| `paper` | 论文笔记 |
| `comparison` | 对比分析 |
| `summary` | 总结/周报 |
| `decision` | 技术决策记录 |

### 状态标签

| 标签 | 说明 |
|------|------|
| `draft` | 草稿，待完善 |
| `review` | 需要复审 |
| `stale` | 可能过时 |
| `wip` | 进行中 |

## 链接规则

- 每页底部必须包含 `## 相关页面` 章节
- 使用 `[[wikilinks]]` 引用其他页面，而非 Markdown 链接
- 尚未创建的页面也可引用（会自动形成“待创建页面”列表）
- 跨领域关联尤其重要，不要局限于当前目录
