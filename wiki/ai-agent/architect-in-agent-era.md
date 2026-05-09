---
title: Agent 时代架构师的核心能力
description: 七项不可绕过的系统能力——上下文管理、工具设计、评估体系、状态持久化、Harness 底座、权限沙箱、多 Agent 边界。分清半衰期长的能力 vs 半衰期短的工具
aliases: [Agent架构师, 架构师能力, Harness半衰期]
tags: [ai-agent, concept, architecture]
sources: [2026/Harness/架构师应该学什么.md]
created: 2026-05-09
updated: 2026-05-09
---

# Agent 时代架构师的核心能力

## 核心判断

> "框架和模型会换，但'给模型设计一个可靠工作环境'的工程能力会留下来。"

分清半衰期：协议、状态、沙箱、评估、工具契约 = 长半衰期（系统能力）；模型封装、CLI 参数、类 Devin 产品 = 短半衰期（工具选择）。

## 七项系统能力

1. **上下文管理**：五层（模型窗口/记忆/索引/元知识/跨会话状态）
2. **工具设计**：工具契约决定 Agent 行为上限
3. **评估体系**：数据化衡量 Agent 质量
4. **状态持久化**：断点续跑的基础
5. **Harness 底座**：错误恢复、权限、编排
6. **权限与沙箱**：生产环境不可或缺
7. **多 Agent 边界**：职责划分与通信

## 起步建议

单 Agent + 5-7 个工具 + 50 条评估样本 + trace，从可度量可回滚的小闭环开始。

> "Agent 产品的质量很难只按模型版本讨论，更准确的发布单元是模型加 Harness 的组合。"
> "在 Agent 领域，filter 比 feed 更重要。"

## 相关页面

- [[agent-harness-overview]] — Harness 综述
- [[agent-architecture-patterns]] — 架构模式
- [[claude-code-harness]] — Claude Code Harness
