---
title: Agent Harness 综述
description: Agent Harness 六承重层（主循环/工具/记忆/状态/权限/验证）拆解——同一模型只换 Harness 结果可差一个量级
aliases: [Harness综述, Agent Harness, The Anatomy of an Agent Harness]
tags: [ai-agent, concept, architecture]
sources: [2026/05/09/Agent-Harness综述.md]
created: 2026-05-09
updated: 2026-05-09
---

# Agent Harness 综述

## 三层概念关系

| 层 | 职责 | 类比 |
|----|------|------|
| Prompt Engineering | 怎么对模型说 | 指令 |
| Context Engineering | 让模型看到什么 | 工作台 |
| Harness Engineering | 系统怎么运行/持久化/验证/兜底 | 操作系统 |

裸 LLM 是一颗没有 RAM、磁盘、I/O 的 CPU——Harness 是让这台机器持续跑起来的一切。

## 六个承重层

### 1. 主循环
心脏。表面是 while loop，难点在谁控制循环、何时终止、出错后怎么回来。

### 2. 工具系统
手。不只注册函数名，还要管参数校验、执行隔离、失败恢复。

### 3. 上下文与记忆
三层记忆：轻量索引→详细主题文件→原始会话记录。原则：记忆是线索不是真相。

### 4. 状态与检查点
系统需知道做到哪、失败从哪恢复。假设必然失败，有 checkpoint 就是恢复问题。

### 5. 权限、错误与安全
模型负责提出动作，系统决定能不能做。高风险动作必须有边界。

### 6. 验证与纠偏
Demo 和生产的分水岭。Claude Code 创始人：让模型验证自己，质量提升 2-3 倍。

## 七步完整循环

组装输入 → 模型推理 → 分类输出 → 执行工具 → 回写结果 → 更新状态 → 决定是否继续

## 为什么 2026 年都在谈 Harness

- 同一模型只换外围，LangChain 从 TerminalBench 前 30 外拉到第 5
- 长任务误差累积：10 步每步 99% → 全链路仅 ~90%
- 2024 卷 Prompt → 2025 补 Context → 2026 收到 Harness

## 设计取舍

模型弱时补结构，模型强后删 dead weight。Manus 半年重建五次，每次做减法。

## AGENTS.md / Spec / Skills 也是 Harness

共同目标：把知识从聊天记录搬出来、把规则从口口相传搬出来、把验证从"我觉得差不多"搬到系统动作里。

> "如果你不是模型，你就在做 Harness。"

## 相关页面

- [[claude-code-harness]] — Claude Code Harness 架构
- [[agent-architecture-patterns]] — Agent 架构模式
- [[claude-code-vs-opencode]] — 框架对比
