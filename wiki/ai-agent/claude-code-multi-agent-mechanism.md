---
title: Claude Code 多Agent实现机制
description: Claude Code 三种多Agent机制（常规Subagent、Fork Subagent、Coordinator模式）的源码级分析——隔离、通信、并发、缓存优化
aliases: [多Agent机制, Multi-Agent实现, Claude Code多Agent]
tags: [ai-agent, practice, architecture]
sources: [2026/05/10/Claude Code 多Agent实现机制.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 多Agent实现机制

**来源**：微信公众号「小林coding」，基于 Claude Code 源码泄漏的深入分析。

## 三种 Multi-Agent 机制

Claude Code 源码中与「多 Agent」相关的代码有三套不同机制：

| 机制 | 模式 | 适用场景 |
|------|------|---------|
| 常规 Subagent | 父子型 | 任务遇到子问题时派 subagent 搞定，拿结果继续 |
| Fork Subagent | 父子型（缓存优化） | 需要继承父 agent 完整上下文但不想污染主循环 |
| Coordinator 模式 | 主从型 | 大任务 + 高并发拆解，主 agent 退化为纯协调者 |

## 为什么需要 Multi-Agent？

单个 agent 处理复杂任务的三个瓶颈：
1. **上下文爆炸** — 多阶段任务内容全塞到同一上下文，token 消耗失控
2. **职责混乱** — 一个 agent 既当研究员又当程序员又当评审员，容易跑偏
3. **无法并发** — 一次只能做一件事

## 架构核心：三个维度的设计

### 1. 隔离机制
按**字段粒度**决策而非整体：读文件缓存克隆副本、写全局状态关闭、任务注册通路保留、深度计数追踪嵌套层级。详见 [[claude-code-subagent]]。

### 2. 通信机制
消息驱动而非函数调用：父→子走消息队列（SendMessage），子→父把完成通知包装成 XML 伪装用户消息（task-notification）。天然异步、天然支持并发。

### 3. Fork Subagent
缓存友好的轻量分身——复用父 agent 的字节级缓存前缀（system prompt、用户上下文、工具池、对话历史），将 subagent 成本降至 10%。详见 [[claude-code-fork-subagent]]。

### 4. Coordinator 模式
主 agent 退化为纯协调者，只做三件事：派 worker、收结果、合成答案。通过 prompt 约束角色分离，通过工具白名单保持扁平结构。详见 [[claude-code-coordinator-mode]]。

## 5 条 Multi-Agent 设计原则

1. **上下文隔离要按字段粒度做** — 每项状态单独决策，不一刀切
2. **通信走消息，不走函数调用** — 异步 + 消息队列，天然支持并发
3. **工具权限要分级管控** — 全局黑名单 → 类型黑名单 → 异步白名单三层过滤
4. **缓存友好是一种架构能力** — 成本优化本身就是能力的一部分
5. **并行优先 + 协调者合成** — 真正的多 agent 威力在并发，协调者必须理解而非转发

## 相关页面

- [[claude-code-subagent]] — 子代理隔离架构（工具隔离 + 上下文隔离）
- [[claude-code-fork-subagent]] — Fork Subagent 缓存优化机制
- [[claude-code-coordinator-mode]] — Coordinator 多 Agent 并行协作
- [[dive-into-claude-code]] — 学术论文：5价值观+13原则+7组件架构
- [[sub-agent-vs-team]] — Sub-Agent vs Agent Team 对比
- [[claude-code-architecture]] — Claude Code 源码架构分析
