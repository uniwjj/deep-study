---
title: Agent 自主规划
description: AI Agent 的自主规划能力——目标分解、路径搜索、动态调整
aliases: [agent-autonomous-planning, Autonomous Planning, 自主规划]
tags: [ai-agent, concept, draft]
sources: [2026/05/10/lint-stub.md]
created: 2026-05-14
updated: 2026-05-14
status: draft
---

# Agent 自主规划

AI Agent 在执行复杂任务时，需要自主将高层目标分解为可执行的子任务序列，并在执行过程中根据反馈动态调整规划。

## 核心能力

- **目标分解**：将模糊任务拆解为结构化的步骤链
- **路径搜索**：在多条可行路径中选择最优执行方案
- **动态重规划**：执行受阻时根据环境反馈调整后续步骤

## 相关范式

- [[agent-design-paradigms]] — ReAct / Plan-and-Execute / Reflection 范式
- [[agent-multi-agent-collaboration]] — 多 Agent 协作中的任务分配

## 相关页面

- [[glm5-harness-practice]] — GLM-5 Harness 中的规划实践
- [[agent-architecture-patterns]] — Agent 架构模式
