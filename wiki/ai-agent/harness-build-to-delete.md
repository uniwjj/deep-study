---
title: Harness Build to Delete
description: Anthropic Harness 第二阶段——从"补"到"删"，模型升级后做 dead weight inventory，三招：用通用工具、还控制权给模型、保留承重边界
aliases: [Build to delete, Harness演进]
tags: [ai-agent, concept]
sources: [2026/05/09/Harness三招从补转删.md]
created: 2026-05-09
updated: 2026-05-09
---

# Harness Build to Delete

## 前后两篇的转变

前篇（长任务）讲"补"：planner/evaluator/context reset/sprint contract 都在替模型兜底。
后篇（Harnessing Claude's Intelligence）问"删"：编排是否还必须由 Harness 接管？

## 三招

1. **先用 Claude 已经熟悉的通用工具** — 减少定制化
2. **把三类控制权还给模型** — 编排权/上下文管理权/记忆选择权
3. **保留真正承重的边界** — 缓存/安全/可观测性

## 核心工程节奏

先承认模型有边界 → Harness 外移为系统能力 → 每次模型升级后检查 → **能删就删**。

> "真正成熟的 Harness，不只补短板，还得在短板消失后主动删掉曾经的补丁。Build to delete."

## 相关页面

- [[agent-harness-overview]] — Harness 综述
- [[claude-code-harness]] — Claude Code Harness
- [[harness-design-long-running]] — 长任务 Harness 设计
