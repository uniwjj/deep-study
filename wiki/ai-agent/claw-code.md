---
title: claw-code
description: AI 编码工具 Harness 系统，Rust 重写版——2 小时破 10 万星，工具编排、任务自动化、Agent 运行时上下文管理
aliases: [claw-code, clawcode, AI编码Harness]
tags: [ai-agent, tool]
sources: [2026/04/02/claw-code突破10万星.md]
created: 2026-05-09
updated: 2026-05-09
---

# claw-code

## 概况

- 上线时间：2026-03-31
- 2 小时内突破 **10 万 Star**，历史最快之一
- Rust 92.9% + Python 7.1%
- 作者：华尔街日报报道的"超级 AI 用户"，去年使用 250 亿 AI 编码 Token

## 定位

AI 编码工具的 **Harness 系统**，核心能力：
- 工具编排与连接
- 任务自动化
- Agent 运行时上下文管理

## 六大核心模块

| 模块 | 功能 |
|------|------|
| api-client | API 客户端，OAuth + 流式传输 |
| runtime | 会话状态与 MCP 编排 |
| tools | 工具定义与执行 |
| commands | 斜杠命令与技能发现 |
| plugins | 插件系统 |
| claw-cli | 交互式 CLI |

## 项目起源

起源于 Claude Code 代码泄露，48 小时 Python 重写 → Rust 重构追求极致性能。

## 功能差距

Rust 重写版并非完整复刻 TypeScript 原版。缺失：插件加载/安装/更新、Hook 执行、远程 Agent、MCP 深度集成（仅 stdio/bootstrap）、Skills 注册（无打包 pipeline）。PARITY.md 明确声明非 feature-parity。

## 相关页面

- [[claude-code]] — Claude Code 开发工具
- [[claude-code-architecture]] — Claude Code 源码架构
- [[agent-harness]] — Agent Harness 概念
