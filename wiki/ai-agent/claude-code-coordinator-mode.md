---
title: Claude Code Coordinator 模式
description: Claude Code 的主从型多Agent并行协作模式——主agent退化为纯协调者，worker并行执行，通过prompt约束角色分离
aliases: [Coordinator模式, Coordinator模式, 协调者模式, 并行多Agent]
tags: [ai-agent, concept, architecture]
sources: [2026/05/10/Claude Code 多Agent实现机制.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code Coordinator 模式

Coordinator 模式是 Claude Code 多 Agent 设计中最「多 Agent」的部分——主 agent 退化为**纯协调者**，通过异步 worker 实现真正的并行协作。

## 启用方式

需要同时满足两个条件：
1. 编译时功能开关 `COORDINATOR_MODE`
2. 运行时环境变量 `CLAUDE_CODE_COORDINATOR_MODE=1`

## 核心设计：角色分离

| 维度 | 常规模式主 agent | Coordinator 模式主 agent |
|------|-----------------|------------------------|
| 角色 | 全能选手（读/写/测/规划都做） | 纯协调者 |
| 职责 | 自己干活 + 偶尔派人 | 只派 worker、收结果、合成答案 |
| 工具 | 全部可用 | 多一套协调专属工具 |

角色转换通过 system prompt 强制约束：

> "You are a **coordinator**. Your job is to direct workers to research, implement and verify code changes, then synthesize results and communicate with the user."

## 协调者工具箱

Coordinator 模式下主 agent 获得一套「团队管理」工具：

| 工具 | 用途 |
|------|------|
| Create Team | 创建 worker 团队 |
| Delete Team | 解散团队 |
| Send Message | 给运行中的 worker 发后续指令 |
| Synthetic Output | 合成最终输出给用户 |
| Stop Worker | 停止跑偏的 worker |

这套工具**不给 worker 使用**，保持扁平结构——worker 不需要再去协调别人。

## 并行执行机制

协调者的 prompt 里核心一句话：

> **Parallelism is your superpower.** Workers are async. Launch independent workers concurrently whenever possible.

核心事实：**派 worker 的工具调用可以在同一条 assistant 消息里出现多次**，底层并发执行：

```
派 worker 调研 auth 模块    ←  同时启动
派 worker 调研 session 模块  ←  同时启动
派 worker 调研 token 模块    ←  同时启动
```

三个 worker 同时干活，协调者等通知一条条返回。

## 任务流水线：四阶段

| 阶段 | 执行者 | 目的 |
|------|--------|------|
| 调研 | Workers（并行） | 调查代码库、找文件、理解问题 |
| **合成** | **协调者本人** | 读完发现、理解问题、写实现规格 |
| 实现 | Workers | 按规格做具体修改、提交 |
| 验证 | Workers | 测试改动是否真的工作 |

**合成阶段协调者必须亲自做**——这是协调者存在的意义。Prompt 反复强调：不要偷懒让 worker "based on your findings, implement the fix"，而是自己把 findings 读懂、写成具体的规格再派下去。

## Continue vs Spawn 决策

遇到新任务，是给老 worker 续命还是派新 worker？

| 情况 | 决策 | 原因 |
|------|------|------|
| 新任务与 worker 已有上下文高度相关 | 续命（Continue） | 已经熟悉相关文件，沟通成本低 |
| 任务与 worker 上下文无关或之前走偏 | 新 worker（Spawn） | 避免旧上下文干扰判断 |
| 验证类工作 | **永远新 worker** | 不能让自己验收自己 |

## 与其他模式对比

| 维度 | 常规 Subagent | Fork Subagent | Coordinator 模式 |
|------|-------------|---------------|-----------------|
| 主 agent 角色 | 全能选手 | 全能选手 | 纯协调者 |
| Subagent 执行 | 同步（2分钟后转后台） | 同步 | 默认异步 |
| 并发程度 | 偶尔并发 | 偶尔并发 | 最大化并发 |
| 系统形态 | 父子树 | 父子（字节复用） | 协调者+worker 扁平层 |
| 缓存优化 | 无 | 有（字节级缓存复用） | 无 |
| 与 Fork 兼容 | ✅ | ✅ | ❌ 互斥 |

## 工程启示

1. **角色分离** — 协调和干活是两件事，不要让同一个 agent 身兼二职
2. **并发优先** — 异步 + 消息队列是并发的基础
3. **合成不转发** — 协调者要理解中间结果，不能当传话筒
4. **扁平不递归** — 通过工具权限把层级限制在两层

## 相关页面

- [[claude-code-multi-agent-mechanism]] — 多 Agent 机制总览
- [[claude-code-subagent]] — 常规子代理架构
- [[claude-code-fork-subagent]] — Fork Subagent 缓存优化
- [[dive-into-claude-code]] — 学术论文分析
- [[sub-agent-vs-team]] — Sub-Agent vs Agent Team 对比
