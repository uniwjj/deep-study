---
title: Harness 作为新后端
description: Agent 系统 Harness 承担后端职责——Worker/Function/Trigger 三概念重译，四项落地动作（能力目录化/上下线/事件触发/trace设计）
aliases: [Harness Backend, Agent后端]
tags: [ai-agent, concept, architecture]
sources: [2026/05/09/Harness新后端.md]
created: 2026-05-09
updated: 2026-05-09
---

# Harness 作为新后端

## 核心公式

`agents² × services` — 1 Agent 接 5 个后端 = 5 条路径；4 Agent 接 5 个系统 = 80 条组合路径。

## 三概念重译

| 原文 | 工程翻译 | Agent 语义 |
|------|---------|----------|
| Worker | 参与者 | 能连接运行时、注册能力的进程 |
| Function | 能力 | 有稳定 ID 的工作单元 |
| Trigger | 入口 | HTTP/队列/cron/状态变化 |

## 四项落地动作

1. 能力目录化（取代静态函数数组）
2. Worker 上下线纳入正常路径
3. 从主动调用扩到事件触发
4. Trace 从协议层设计

## 六个工程边界

工具目录升级为运行时能力目录。后端开始给 Agent 提供可发现的能力、可恢复的状态、可审计的权限、可追踪的链路。

## 相关页面

- [[agent-harness-overview]] — Harness 综述
- [[harness-engineering-practice]] — 工程实践
