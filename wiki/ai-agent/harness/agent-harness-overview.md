---
title: Agent Harness 综述
description: Agent Harness 六承重层（主循环/工具/记忆/状态/权限/验证）拆解——同一模型只换 Harness 结果可差一个量级；补充 WorkBuddy 五层架构（运行环境/引导/反馈/编排/迭代）与驾驭/约束/整合三能力模型
aliases: [Harness综述, Agent Harness, The Anatomy of an Agent Harness]
tags: [ai-agent, concept, architecture]
sources: [2026/05/09/Agent-Harness综述同一模型体感差异在哪.md, 2026/07/10/从模型到Harness：WorkBuddy如何把Agent做成可用产品.html]
created: 2026-05-09
updated: 2026-07-10
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

## WorkBuddy 五层Harness模型

> 来源：[[workbuddy-agent-product-design|WorkBuddy Agent 产品设计]]，2026年7月

WorkBuddy 将 Harness 定义为一个**控制系统**，核心机制是**前馈（Feedforward）+ 反馈（Feedback）**：

- **前馈**：Agent 行动前提供目标、规则、环境和可用能力，提高首次正确率
- **反馈**：Agent 行动后观察结果，把错误和修正信息返回给 Agent，让问题在进入人工审查前先自我纠正

### 三类能力（从词源拆解）

| 能力 | 含义 | Agent 对应 |
|------|------|-----------|
| 驾驭（Steer） | 控制执行方向、速度和停止时机 | System Prompt/规则文件、Skills、Task/Todo |
| 约束（Constrain） | 防止执行超出安全范围 | 权限边界、Sandbox、Approval Gate、Allowlist/Denylist |
| 整合（Integrate） | 把各项能力配齐并协同 | 执行能力、状态承载、协作机制、自动化 |

### 计算型 vs 推断型控制

| 类型 | 特点 | 示例 |
|------|------|------|
| 计算型 | 确定性程序执行，快、便宜、可重复 | LSP、类型检查、linter、单元测试 |
| 推断型 | 模型语义判断，慢、贵、不确定 | Review Agent、架构审查、AI judge |

**原则：能用计算型信号解决的，优先交给确定性程序；需要语义判断的，再交给审查 Agent。**

### 五层结构

| 层 | 职责 | 核心机制 |
|----|------|---------|
| **1. 运行环境层** | Agent 在哪里执行 | 文件系统、Shell/Bash、Sandbox、Browser、MCP/Connectors、权限边界 |
| **2. 引导层（Feedforward）** | Agent 开始前掌握什么 | 项目上下文、环境上下文、规则与风格、Prompt Cache 策略 |
| **3. 反馈层（Feedback）** | Agent 执行后如何获知错误 | 工具结果含可纠正信息、时间戳校验、外部验证信号回传、Audit Log |
| **4. 编排层** | 多个能力如何组织 | 渐进式加载、意图识别和路由、多模型路由、Teams 多 Agent 协作 |
| **5. 迭代层** | Harness 自身如何持续调整 | 随模型能力精简上下文、根据重复反馈增加机制（需证据支撑） |

反馈按时机分层：快速检查左移到编辑后/提交前；昂贵的架构审查和端到端验证放到集成前后；持续漂移检测交给周期性传感器。

### 使用者视角四类组件

WorkBuddy 团队自身的实践可以归入四类：

| 组件 | 作用 | WorkBuddy 实践 |
|------|------|---------------|
| 上下文工程 | 让 Agent 获得任务所需信息 | 分层规则文件、OpenSpec、Skills、Slash 命令 |
| 架构约束 | 把规则变成可执行检查 | 本地检查、Git Hooks、CI 门禁、审查 Agent |
| 反馈循环 | 把验证结果返回给 Agent | Post-edit checkpoint、CI 结果回传、/team:mr 工作流 |
| 熵管理 | 处理规则/代码/状态的漂移 | 周期性扫描一致性和重复实现 |

这四类组件形成一个持续过程：上下文工程提供规则和任务信息 → 架构约束阻止已知违规 → 反馈循环帮 Agent 修正本次执行 → 熵管理处理跨任务积累的漂移。

## 相关页面

- [[claude-code-harness]] — Claude Code Harness 架构
- [[agent-architecture-patterns]] — Agent 架构模式
- [[claude-code-large-codebase]] — 大型代码库中的 Harness 构建顺序与配置模式
- [[claude-code-vs-opencode]] — 框架对比
- [[workbuddy-agent-product-design]] — WorkBuddy 五层 Harness 完整实现（含驾驭/约束/整合三能力）
- [[workbuddy-context-engineering]] — WorkBuddy 上下文工程实践
- [[harness-engineering-practice]] — Harness Engineering 实践（Human-first → Agent-aware）
