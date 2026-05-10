---
title: Claude Code Swarm 多Agent协作
description: Claude Code Swarm 系统——对等 Teammate 协作、Mailbox 文件级通信、Auto-Resume、Coordinator 模式扩展细节
aliases: [Swarm, Teammate, Mailbox, 多Agent团队, 对等协作]
tags: [ai-agent, architecture, concept]
sources: [2026-05-10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code Swarm 多Agent协作

Swarm 是 Coordinator 模式的对等替代方案——多个 Claude Code 实例组成 team，由 leader 通过消息传递协调。

## In-Process Teammate

运行在**同一个 Node.js 进程**中：

| 特性 | 实现 |
|------|------|
| 隔离 | AsyncLocalStorage |
| 消息上限 | 50 条（内存安全保护） |
| 空闲机制 | `isIdle` 标志启用 work-stealing |
| 重定向 | `currentWorkAbortController` 可在不杀死 teammate 的情况下 redirect |

## Mailbox（文件级通信）

基于文件的消息存储：

- 消息写入接收方的磁盘 mailbox 文件
- 持久化、可检查、零成本（不需要消息中间件）
- 为一个 session 中几十条消息的量级设计

## SendMessage — 通用通信原语

四种路由模式：

| 路由 | 路径 |
|------|------|
| Bridge messages | 通过 Anthropic Remote Control 服务器跨机器通信 |
| UDS messages | 通过 Unix Domain Socket 本地进程间通信 |
| In-process subagent routing | 最常见路径，自动 resume |
| Team mailbox | 当 team 上下文激活时的后备路径 |

### Auto-Resume 模式

当 SendMessage 目标是一个已完成或被杀的 agent 时，不返回错误，而是**从磁盘转录中复活该 agent**。对 coordinator 来说：向死 agent 发消息和向活 agent 发消息是同一个操作。

## Task 状态机

七种 task 类型，五种状态，无环图：

```
pending → running → completed | failed | killed
```

### Task 类型

| 类型 | 前缀 | 描述 |
|------|------|------|
| `local_bash` | b | 后台 shell 命令 |
| `local_agent` | a | 后台子 agent |
| `remote_agent` | r | 远程会话 |
| `in_process_teammate` | t | Swarm teammate |
| `local_workflow` | w | Workflow 脚本执行 |
| `monitor_mcp` | m | MCP 服务器监控 |
| `dream` | d | 推测性后台思考 |

### 基础状态字段

每个 task：`id`, `type`, `status`, `description`, `toolUseId`, `startTime`, `endTime`, `totalPausedMs`, `outputFile`, `outputOffset`, `notified`

`outputFile` 是异步执行与父 agent 对话之间的桥梁。`notified` 防止重复的完成消息。

## 后台通信三通道

1. **磁盘 outputFile**：每个 task 写入输出文件，父 agent 通过 `outputOffset` 逐增量读取
2. **Task 通知**：agent 完成后以伪装成 user 消息的 XML 通知注入对话
3. **命令队列**（`pendingMessages` 数组）：在 tool-round 边界清空

## Coordinator 模式补充细节

Coordinator 只获得 3 个工具：**Agent、SendMessage、TaskStop**。不能读文件、不能编辑代码、不能执行 shell 命令。

370 行 system prompt 中的四个教义：
1. **永远不要代理理解力**——coordinator 必须包含文件路径、行号、确切改动
2. **并行是你的超能力**——只读任务自由并行；写密集任务按文件集序列化
3. **四阶段流程**：Research → Synthesis → Implementation → Verification
4. **续命 vs 生成新 worker**：取决于上下文重叠程度

### Scratchpad（协作缓冲区）

Coordinator 和 worker 通过共享文件系统目录交换信息，无需权限弹窗。按引用传递而非按值。

## 模式选择决策矩阵

| 场景 | 模式 | 原因 |
|------|------|------|
| 编辑时同时跑测试 | Simple delegation | 单后台 task，无需协调 |
| 跨 3 个模块重构 40 个文件 | Coordinator | Research → synthesis → 并行执行 |
| 多天 feature 开发 | Swarm | 长生命周期 agent、plan 审批、对等通信 |
| 已知位置的 bug 修复 | 单 agent | 编排开销超过收益 |

## 相关页面

- [[claude-code-multi-agent-mechanism]] — 三种多 Agent 机制总览
- [[claude-code-subagent]] — 子代理委托架构
- [[claude-code-fork-subagent]] — Fork Subagent 缓存优化
- [[claude-code-coordinator-mode]] — Coordinator 并行协作模式
- [[claude-code-session-persistence]] — 会话持久化（Auto-Resume 依赖）
- [[claude-code-remote-execution]] — 远程执行（Bridge 消息路由）
