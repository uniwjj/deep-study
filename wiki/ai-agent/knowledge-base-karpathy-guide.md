---
title: Karpathy 知识库搭建指南
description: 基于 Andrej Karpathy 理念的 LLM 知识库四步法——Obsidian + LLM Agent 将原始资料编译为结构化 wiki，定期健康检查与增量更新
aliases: [LLM知识库教程, 知识库四步法]
tags: [ai-agent, practice]
sources: [2026/04/05/LLM搭建知识库保姆级教程.md]
created: 2026-05-09
updated: 2026-05-09
---

# Karpathy 知识库搭建指南

## 四步法

```
收集原始资料 → 编译成 wiki → 提问 & 输出 → 定期检查更新
```

来源：Andrej Karpathy

## 目录结构

```
Vault/
├── raw/      ← 原始文件（不可改）
├── wiki/     ← LLM 自动生成（不可手动改）
├── output/   ← 提问生成的输出
├── images/   ← 图片
└── Claude/   ← 知识库规则配置
```

## 步骤

### 1. 收集原始资料
- 工具：Obsidian + Web Clipper 插件
- Web Clipper 设置保存地址为 raw 文件夹
- 采集的内容：网页文章、笔记、心得等

### 2. 编译成 wiki
- LLM Agent（Claude Code/Cursor 等）读取 raw/
- 自动生成结构化 wiki Markdown 文件
- wiki 类似百科：知识点 + 相关链接集中在一个文件中

### 3. 提问 & 输出
- 基于编译好的 wiki 提问
- 输出到 output/：Markdown、幻灯片等

### 4. 定期检查更新（/lint）
每周让 LLM 执行健康检查：
1. 找出重复或矛盾的概念
2. 发现应互链但未链的概念
3. 建议新增或补充的页面
4. 更新 INDEX.md
5. raw/ 有新内容 → 增量编译

## 核心工具

- **Obsidian**：本地 Markdown 知识管理，关系图谱可视化
- **LLM Agent**：编译、回答、健康检查
- **Web Clipper**：网页内容采集

## 核心理念

> "在用 AI 办公的年代，知识库的丰富程度决定你和别人的差距。"

## 相关页面

- [[obsidian-second-brain]] — Obsidian 个人第二大脑
- [[obsidian-claudian]] — Obsidian + Claude Code 方案
- [[llm-wiki]] — LLM Wiki 知识库理念
- [[ai-knowledge-base]] — AI 知识库方案对比
