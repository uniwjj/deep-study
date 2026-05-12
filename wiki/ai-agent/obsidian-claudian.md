---
title: Obsidian + Claudian
description: Obsidian 本地知识库 + Claude Code + Claudian 插件的 AI 第二大脑方案，从 Prompt → Agent → Skill 的三级进化
aliases: [Claudian, Obsidian AI知识库, AI第二大脑]
tags: [ai-agent, tool, practice]
sources: [2026/02/26/Obsidian + Claudian真香警告：AI知识库终极方案.md]
created: 2026-05-09
updated: 2026-05-09
---

# Obsidian + Claudian

## 系统架构

```
Obsidian 文档库（本地存储+云同步） ←→ 文档编辑区 ←→ AI 工作区（Claude Code + Claudian 插件）
       ↕
  Skills 能力层（md 格式，可装卸）
```

三个窗口协同：左侧文档库导航、中间编辑、右侧 AI 对话。核心能力：在 AI 工作区发指令即可完成选题、头脑风暴、边写边改、去 AI 味等全流程。

## 四大优点

### 1. 文件自主权
- Obsidian 本地存储 + 云同步，不被任何平台绑定
- 与 NotebookLM/ChatGPT 不同，数据不归属特定厂商
- "一辈子只需要建一次'数字分身'"

### 2. 跨模型记忆
- 记忆存在本地文件系统，不与特定模型绑定
- 可随时切换底层模型（国产 Minimax、智谱 GLM 等）
- "谁便宜好用，咱们就换谁"

### 3. Skills 能力接入
```
Prompt（自己干活+AI加速） → Agent（代理执行） → Skill（雇人干活）
```
- Skill 是 md 文档，标准 txt 格式，可自行优化或"直接开除"
- Obsidian 文档库窗口比命令行对非开发者更友好

### 4. Obsidian 知识图谱
- 标签 + wikilinks 双向链接自动形成知识图谱
- AI 自动整理比手动快 100 倍
- 能高效找回丢失的记忆（如 AI 回忆特定经历），生成不雷同的文章

## 部署步骤

1. 安装 Obsidian（免费开源）
2. 安装 Claude Code（`brew install node && npm install -g @anthropic-ai/claude-code`）
3. 安装 Claudian 插件（第三方公开插件）
4. 接入大模型（设置 `ANTHROPIC_BASE_URL` / `ANTHROPIC_API_KEY` / `ANTHROPIC_MODEL`）
5. 导入历史数据（微博用 weibo-achiever 油猴插件 + python 脚本；知乎用 zhihu to markdown 插件）
6. 配置 PARA 文件夹架构
7. 配置记忆系统（不配置"就像一个只有 7 秒记忆的人"）
8. 配置改进系统（self-improving skill）
9. 思维方式从 Prompt 到 Skill 进化

**作者判断**：对文字工作者来说，Obsidian + Claudian 可能是最佳实践方式。作者花 4 天踩坑，指南帮助读者 1 天搞定。

## 相关页面

- [[llm-wiki]] — LLM Wiki 知识库理念（Karpathy）
- obsidian-para — PARA 文件夹分类架构
- [[agent-skills-system]] — Agent Skills 能力体系
- [[ai-governance]] — 跨模型记忆与数据主权
