---
title: Claude Code Harness 架构
description: Claude Code 的 Harness 编排系统——入口适配+会话成型+运行桥接，三层架构（入口分流→Harness编排→Runtime支撑），Agent Loop + 状态管理 + 记忆系统 + 工具系统
aliases: [Harness架构, Agent Harness, Claude Code编排层]
tags: [ai-agent, architecture, concept]
sources: [2026/04/03/Claude Code Harness架构设计.md]
created: 2026-05-09
updated: 2026-05-09
---

# Claude Code Harness 架构

## 定义

Harness 是一套「入口适配 + 会话成型 + 运行桥接」编排系统，本质是把多入口、多模式、多运行位置收敛成统一的 **agent turn 执行模型**。

## 三层架构

```
入口分流层 → Harness 会话编排层 → Runtime 与支撑层
```

| 层 | 职责 | 核心组件 |
|----|------|----------|
| **入口分流** | CLI/交互/SDK/assistant/远端 → 统一路由 | main.tsx 参数解析+模式判断 |
| **Harness 编排** | 收敛为统一 turn 契约 | 交互会话/无界面会话/远端接入 + 工具能力+状态持久化 |
| **Runtime 支撑** | 执行 Agent Loop | 本地 Runtime / 远端 bridge / server |

## Agent Loop

核心设计：「将智能下沉给模型，释放智能自主权」

Harness 回答"请求从哪来、交给谁"；Agent Loop 回答"链路如何运行"。

## 状态管理

三层分层协同：

```
会话运行态 → 全局会话态 → 持久化层
```

确保长会话稳定性与上下文恢复能力。

## 记忆系统

四层管理：

| 层 | 范围 | 说明 |
|----|------|------|
| L1 | 当前会话沉淀 | 本次会话内积累 |
| L2 | 项目/团队记忆 | 共享的领域知识 |
| L3 | Agent 专属记忆 | Agent 个人学习 |
| L4 | 动态注入 | 每轮按相关性动态注入 |

记忆层在 runtime 中执行筛选 → 注入 → 回写流程。

## 工具系统

内置工具、MCP 外部能力、skills/plugins 扩展能力 → 运行时统一为一致调用接口。

## 设计哲学

**「下限可控、上限可拓」**——技术栈和架构形态可以相似，真正拉开差距的是细节打磨。

## 相关页面

- [[claude-code-architecture]] — Claude Code 源码架构五层主线
- [[claude-code-auto-dream]] — 记忆整合机制
- [[claude-code-hidden-features]] — 87 个未发布功能
- [[agent-architecture-patterns]] — Agent 架构模式
- [[openclaw]] — OpenClaw 对比
