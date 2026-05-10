---
title: DeerFlow 2.0
description: 字节跳动开源 Super Agent Harness——组装厂模式（LangGraph 编排+Claude Code 推理+强制沙箱），支持多模型，可组合架构
aliases: [DeerFlow, 字节跳动Agent]
tags: [ai-agent, tool]
sources: [2026/05/09/DeerFlow.md]
created: 2026-05-09
updated: 2026-05-09
---

# DeerFlow 2.0

## 定位

Deep Exploration and Efficient Research Flow。不是 Agent 框架，是 Agent 的**组装厂**。

## 架构

```
DeerFlow
├── Skills & Tools
├── Sub-Agents
├── Sandbox（强制）
├── Memory
└── Claude Code Integration
```

LangGraph 做编排，Claude Code 做推理引擎（`supports_thinking: true`），支持 Doubao-Seed-2.0/DeepSeek v3.2/GPT-4/Claude Sonnet 4.6。

## 相关页面

- [[agent-vs-framework-vs-harness]] — 三者区别
- [[claude-managed-agents]] — CMA
