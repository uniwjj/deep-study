---
title: Harness Engineering 实践
description: Human-first → Agent-aware 工程系统转型——三层能力（系统可读性/防御机制/自动化反馈），Stripe 实践案例
aliases: [Harness实践, Agent-aware系统]
tags: [ai-agent, practice, concept]
sources: [2026/05/09/Harness工程实践.md]
created: 2026-05-09
updated: 2026-05-09
---

# Harness Engineering 实践

## 问题本质

从 **Human-first system** → **Agent-aware system**。工程体系围绕人类构建，AI 不具备隐性知识。问题不在模型，在工程系统。

## 三层能力模型

```
系统可读性 ─── 防御机制 ─── 自动化反馈
      ▲                      │
      └──────────────────────┘
```

| 层 | 核心 |
|----|------|
| **系统可读性** | 隐性知识显性化，逐步披露上下文，工具暴露为 CLI/API |
| **防御机制** | 收敛 AI 行动空间——静态分析/测试/架构验证 = 系统边界的物理定律 |
| **自动化反馈** | 每次修改立即反馈：开发环境→CI→线上监控闭环 |

## 核心理念

AI Coding 的突破往往来自工程系统而非模型能力。

Stripe 案例 + Routa.js（多 Agent 协作遵循统一规范）：github.com/phodal/routa

## 相关页面

- [[agent-harness-overview]] — Harness 综述
- [[harness-design-long-running]] — 长任务设计
