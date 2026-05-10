---
title: Sub-Agent vs Agent Team
description: 多 Agent 两种核心协作模式对比——Sub-Agent 隔离执行 vs Agent Team 实时协作，选型铁律：独立用 Sub-Agent，依赖用 Team
aliases: [多Agent架构, Sub-Agent, Agent Team]
tags: [ai-agent, concept, architecture]
sources: [2026/05/09/Sub-Agent vs Team.md]
created: 2026-05-09
updated: 2026-05-09
---

# Sub-Agent vs Agent Team

## 核心区别

| | Sub-Agent | Agent Team |
|---|---|---|
| **本质** | 隔离执行 | 实时协作 |
| **通信** | 不能互相通信，经父级流转 | 实时双向通信 |
| **上下文** | 完全隔离 | 共享 |
| **适用** | 任务完全独立 | 任务互相依赖 |

## Sub-Agent 三条铁律

1. Agent 间不能互相通信
2. 不能自建新 Agent
3. 信息流必须经过父级

## Agent Team

主导 Agent 分配任务 + 成员执行 + 共享任务层跟踪进度。

核心：前端改 API → 后端立刻感知同步，无需人工介入。

## 选型铁律

独立 → Sub-Agent，依赖 → Team。

**最常见的错误**：按角色拆分（Planner → Developer → Tester），应按**上下文边界**分解。

## 五种实用模式

提示链 → 路由 → 并行化 → 编排-工作器 → 评估-优化（覆盖 90%+ 场景）

> "删除一个 Agent 往往比增加一个 Agent 更值钱。"

## 相关页面

- [[agent-harness-overview]] — Harness 综述
- [[claude-code-vs-opencode]] — SubAgent/Fork/Teammate
- [[openclaw]] — OpenClaw SubAgent + Routed Agent
- [[hermes-agent]] — Hermes PLUR 经验传播
- [[agent-multi-agent-collaboration]] — 多 Agent 协作

## 三框架多 Agent 对比

| 维度 | OpenClaw | Claude Code | Hermes |
|------|---------|-------------|--------|
| SubAgent 通信 | 只向主 Agent 汇报 | 只向主 Agent 汇报 | 文件 + Skills 协调 |
| 高级模式 | Routed Agent（Gateway 层） | Agent Teams（P2P） | Skills 共享知识层 |
| 独特 | 非阻塞派发 | P2P 直接通信 | PLUR 双向传播 |
