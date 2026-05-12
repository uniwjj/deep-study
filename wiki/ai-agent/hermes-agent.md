---
title: Hermes Agent
description: Nous Research 开源自改进 AI Agent 框架——内置学习闭环、自动技能进化、持久多层记忆、40+ 工具，越用越像你的数字分身
aliases: [Hermes, Hermes Agent, 自进化Agent]
tags: [ai-agent, tool]
sources: [2026/04/08/Hermes Agent教程.md]
created: 2026-05-09
updated: 2026-05-09
---

# Hermes Agent

## 核心创新：内置学习闭环

每次完成任务后自动提炼可复用 Skill 存入持久记忆。

## 五大特性

- **持久多层记忆**：SQLite + FTS5 + LLM 自动总结，跨会话永久记忆
- **自动技能进化**：任务完成后生成 Markdown Skill，下次直接调用并迭代
- **自主执行**：40+ 工具（终端/浏览器/文件/代码/搜索）
- **模型灵活**：OpenRouter 200+ 模型 + OpenAI + Anthropic + Ollama
- **MIT 开源**：$5 VPS 即可运行

## Hermes vs OpenClaw

| | Hermes | OpenClaw |
|---|---|---|
| **设计哲学** | 自我进化，自动成长 | 万能工具箱，手动喂 Skill |
| **记忆** | 分层召回，跨会话更懂你 | 全量上下文 |
| **Token** | 节省 30%-60% | 全量消耗 |
| **迁移** | `hermes claw migrate` | — |

两者可互补：OpenClaw 做执行，Hermes 做大脑。

快速安装：`curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash`

## 相关页面

- [[openclaw]] — OpenClaw 数字员工
- [[agent-skills-system]] — Agent Skills 体系
- [[agent-memory-system]] — Agent 记忆系统
