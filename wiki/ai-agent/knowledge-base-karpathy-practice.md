---
title: Karpathy 知识库实操指南
description: 五步从零搭建个人 AI 知识库——三文件夹 + CLAUDE.md 说明书，agent-browser 自动采集，无数据库无服务器
aliases: [Karpathy知识库搭建]
tags: [ai-agent, practice]
sources: [2026/04/06/跟着Karpathy从零搭建AI知识库.md]
created: 2026-05-09
updated: 2026-05-09
---

# Karpathy 知识库实操指南

## 核心公式

**文件夹 + 文本文件 + AI**，无需数据库、无向量嵌入、无服务器。

## 三文件夹结构

```
my-knowledge-base/
├── raw/      # 源材料，什么都往里扔（不整理不重命名）
├── wiki/     # AI 整理的百科
└── outputs/  # 生成的答案和报告
```

## 五步搭建

1. 创建三个文件夹
2. 把文章/笔记/论文/截图扔进 raw/（"超级简单，完全扁平" — Karpathy）
3. 自动化采集：`agent-browser`（Vercel Labs，26K+ stars，比 Playwright MCP 省 82% token）
4. 写 CLAUDE.md 说明书（最关键但最易被跳过）
5. 一条指令编译：`读取 raw/ 的所有内容，按 CLAUDE.md 规则编译 wiki`

## CLAUDE.md 核心要素

- 文件夹分工说明
- Wiki 规则：每主题一个 .md、`topic-name` 链接、维护 INDEX.md
- 个人兴趣点列表

## 为什么有效

- 降低认知负担：先存储，AI 帮整理
- AI 擅长建立海量信息关联（人脑的短板）
- 越用越好用：更多内容 → 更多关联 → 更深洞察

## 适用/不适用

- ✅ 研究人员、内容创作者、终身学习者、项目经理
- ❌ 追求即时满足者、已有成熟知识体系者、只需简单笔记者

## 进阶

- 每周 15 分钟维护
- 结合 Obsidian 图谱、Zotero 论文导入、Readwise 高亮同步

## 相关页面

- [[knowledge-base-karpathy-guide]] — Karpathy 四步法理论
- [[obsidian-second-brain]] — Obsidian 第二大脑
- [[obsidian-claudian]] — Obsidian + Claude Code
- [[llm-wiki]] — LLM Wiki 理念
