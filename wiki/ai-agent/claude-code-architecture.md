---
title: Claude Code 源码架构
description: Claude Code 源码架构解析——启动入口、Prompt 装配层、工具契约、权限决策管道、长任务续航五层主线
aliases: [Claude Code架构, Claude Code源码解析]
tags: [ai-agent, tool, architecture]
sources: [2026/04/01/Claude Code 源码架构解析.md]
created: 2026-05-09
updated: 2026-05-09
---

# Claude Code 源码架构

## 五层主线

泄露的是工程实现层（非模型权重），最值得看的主线：

```
启动入口 → Prompt装配 → 工具契约 → 权限管道 → 长任务续航
```

## 1. 启动与入口（`src/main.tsx`）

从一开始按**长期交互系统**优化，而非一次性 CLI。这解释了 Claude Code 不只是终端聊天工具。

## 2. Prompt 装配（`src/constants/prompts.ts` + `src/utils/queryContext.ts`）

Prompt 不是静态文案，而是**一组可缓存的装配层**。比许多产品更早把"上下文装配"当成正式工程对象。

```
┌─────────────────────────────┐
│  Prompt 装配（上下文作为可缓存对象）  │
├─────────────────────────────┤
│  prompts.ts   →  静态模板   │
│  queryContext.ts → 动态装配  │
│  cache        →  可缓存     │
└─────────────────────────────┘
```

## 3. 工具契约（`src/Tool.ts` + `src/tools.ts` + `src/tools/FileEditTool/*`）

Anthropic 把"先读再改"和"默认保守"写成机制：
- 工具调用前先把边界写清楚
- 靠的是**工程纪律**，不是神奇技巧
- 运行主链路：`QueryEngine.ts → processUserInput.ts → query.ts`

## 4. 权限管道（`src/utils/permissions/permissions.ts`）

更像**权限决策管道**，不是"最后再补"的界面逻辑：
- 本地环境持续运行的产品，权限不能继续作为界面逻辑存在
- 决策管道：审批规则 → 宽松/严格策略 → 默认拒绝

## 5. 长任务续航

把"历史压缩"和"状态续写"**分开处理**：
- 工程完整性扎实
- 做到最小侵入处理
- 历史旧段 → 摘要压缩 + 状态续写 → 保持上下文连贯

## 架构洞察

- Prompt 从静态文案进阶为可缓存的上下文装配层
- 工具系统遵循"先读再改、默认保守"的工程纪律
- 权限系统本质是决策管道而非界面逻辑
- 长任务续航将压缩与续写分离，保证工程完整性

## 相关页面

- [[claude-code-sourcemap-reverse]] — Sourcemap 逆向还原 1,884 源文件
- [[claude-code-auto-dream]] — 自动梦境记忆整合机制
- [[agent-architecture-patterns]] — Agent 架构模式
- [[ai-agent-security]] — AI Agent 安全与权限
