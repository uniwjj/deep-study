---
title: Claude Code 搜索策略
description: Claude Code 的搜索与检索策略——为什么 51 万行代码中零向量数据库，grep 为何打败 RAG，GrepTool/Glob/Read 的实现细节
aliases: [Search Strategy, grep vs RAG, 搜索策略, 检索策略]
tags: [ai-agent, architecture, concept]
sources: [2026/05/10/Claude-Code-Source-Analysis.pdf]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 搜索策略

Claude Code 源码中最具反直觉的工程决策之一：**512,920 行 TypeScript 中，`vector`、`semantic search` 匹配数为 0，`embedding` 仅为普通英文用法。整个搜索系统不到 1500 行代码。零向量数据库。**

## 四个不使用 RAG/向量数据库的理由

### 1. LLM 已足够聪明

LLM 本身就是最强的理解引擎。grep 返回原始文本，LLM 自己去理解——不需要搜索引擎代为筛选排序。搜索引擎的排序算法基于统计相关性而非语义理解，在代码场景下反而引入噪音。

### 2. grep 的精确性被低估

- grep 是 **100% 确定性**：有就是有，没有就是没有
- RAG 是**概率性的**：可能漏掉关键文件，可能返回不相关内容
- 在代码搜索场景中，精确匹配 > 语义模糊匹配
- 符号名、函数名、错误信息——grep 搜索零误报、零延迟

### 3. 维护成本差距巨大

| | grep | RAG/向量数据库 |
|---|------|--------------|
| 索引维护 | 零 | 每次代码变更需 re-chunk + re-embed |
| 基础设施 | 无 | 嵌入模型 + 向量数据库 |
| 一致性 | 实时 | 异步索引有延迟 |
| 成本 | 零 | 嵌入 API 费用 + 存储费用 |

### 4. 大上下文窗口让暴力搜索可行

100K-1M token 上下文窗口意味着一次性把 grep 结果全塞给模型阅读是可行的。RAG 的设计前提（上下文窗口小，必须预先筛选）已不再成立。

## GrepTool：核心搜索工具

底层使用 **ripgrep**（比 GNU grep 快一个数量级）：

| 特性 | 说明 |
|------|------|
| 3 种 output_mode | `files_with_matches`（默认，省 token）、`content`（精确匹配行）、`count`（统计） |
| head_limit | 默认 **250 行**，精算平衡点防止上下文膨胀 |
| 正则引擎 | PCRE2，支持丰富正则语法 |
| 文件过滤 | `.gitignore` 自动遵守 |

`files_with_matches` 模式是默认值——先让模型看到哪些文件匹配，再决定读哪个。这比一次性返回所有结果省大量 token。

## Glob：文件发现

- 底层 `rg --files --glob`，按 **mtime 升序**排列
- 最多 100 个文件
- GrepTool 的 `files_with_matches` 模式做应用层降序（最近修改在前）
- 模型倾向于先看最近修改过的文件（符合直觉）

## Read：多模态读取

Read 工具远超简单文本读取：

- **图片**：base64 编码，多模态模型直接理解
- **PDF**：`extractPDFPages()` 自动分页
- **Jupyter Notebook**：`readNotebook()` 结构化读取
- **目录**：fallback 到 `ls` 命令

### FILE_UNCHANGED_STUB

重复读取同一文件且内容未变化时，Read 返回 `FILE_UNCHANGED_STUB` 占位符而非完整内容。在长对话中避免了大量冗余 token 消耗。

## 搜索工具组合策略

```
Glob 找到文件 → Grep files_with_matches 缩小范围 → Read 精读
```

三步组合覆盖了从粗到细的完整搜索需求。没有索引、没有嵌入、没有检索增强——三个工具，不到 1500 行代码。

## 设计启示

> "不是所有问题都需要最先进的技术。在代码搜索这个场景中，简单工具 + 聪明模型 > 复杂工具 + 弱模型。"

Anthropic 的决策逻辑：与其花精力维护一个嵌入索引，不如让模型变得更聪明。这笔账算下来，投资模型能力比投资检索基础设施更划算。

## 相关页面

- [[claude-code-tool-system]] — 完整工具系统
- [[claude-code-api-layer]] — Prompt cache 让大上下文经济可行
- [[claude-code-agent-loop]] — 搜索工具在循环中的位置
- [[claude-code-concurrent-tools]] — Read/Grep/Glob 始终并发安全
