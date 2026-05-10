---
title: Agent vs Framework vs Harness
description: 三层概念区别——Agent 干活的智能体、Framework 造智能体的零件库、Harness 管理智能体的操作系统，2026 瓶颈从智能转向可靠性
aliases: [Agent三者区别, 智能体概念层级]
tags: [ai-agent, concept]
sources: [2026/05/09/Agent三层区别.md]
created: 2026-05-09
updated: 2026-05-09
---

# Agent vs Framework vs Harness

## 三层架构

| 层 | 定位 | 职责 |
|----|------|------|
| **Agent** | 干活的智能体 | 思考、规划、调用工具、完成任务 |
| **Framework** | 造智能体的零件库 | 工具调用封装、记忆模块、ReAct 循环 |
| **Harness** | 管理智能体的操作系统 | 任务拆解、上下文管理、状态持久化、漂移纠正、失败重试 |

## 核心判断

- Agent 很聪明，但不稳定、不可控
- Framework 只管"能不能跑起来"，不管"跑得稳不稳"
- Harness 决定 Agent 能否上生产

> "2026 年 AI 竞争的真正瓶颈不再是'智能'而是'可靠性'——拼模型不够，拼 Agent 只是开始，真正拼的是 Harness。"

## 相关页面

- [[agent-harness-overview]] — Harness 综述
- [[harness-engineering-practice]] — 实践总结
