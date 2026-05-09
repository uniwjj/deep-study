---
title: Claude Managed Agents
description: Anthropic 托管 Agent 基础设施——脑手分离架构（Session/Harness/Sandbox），四层 API 抽象，Agent 上线从数月压缩到数天
aliases: [CMA, Managed Agents, 脑手分离, 托管Agent]
tags: [ai-agent, tool, architecture]
sources: [2026/04/09/效率提升十倍.md, 2026/04/09/脑手分离架构拆解.md, 2026/04/09/CMA托管时代.md, 2026/04/09/开发Agent到头了.md, 2026/04/09/重新思考Agent架构.md]
created: 2026-05-09
updated: 2026-05-09
---

# Claude Managed Agents

## 核心问题

从 Agent 原型到生产需要搞定沙箱、状态管理、凭证管理、权限控制、全链路追踪——可能耗费数月。Managed Agents 把这些基础设施全接过去。

## 脑手分离架构

```
Session（会话日志） → Harness（大脑/编排引擎） → Sandbox（双手/隔离执行环境）
```

| 组件 | 角色 | 特性 |
|------|------|------|
| **Session** | append-only 不可变事件日志 | 独立于运行时，持久化，`emitEvent/getEvents` |
| **Harness** | 调用模型+路由工具的无状态组件 | 崩溃后通过 `wake(sessionId)` 从 Session 重建 |
| **Sandbox** | 隔离执行环境 | 通过 `execute(name, input)→string` 统一接口 |

## 设计哲学

接口稳定，底层实现自由替换——类似 OS 将硬件抽象为"进程"和"文件"。

性能：p50 TTFT 降 60%，p95 降超 90%。

## 四层 API 抽象

- **Agent**：声明式配置（模型/系统提示/工具集/MCP）
- **Environment**：容器模板（预装包/网络规则）
- **Session**：运行实例（绑定 Agent+Environment）
- **Events**：SSE 流式通信

8 种语言 SDK + CLI ant。额外费用：$0.08/session-hour。

## 安全

凭证永远不出现在沙箱中。沙箱崩溃可自动恢复，坏了一头换一头。

## 限制

Beta 阶段，只支持 Claude 模型，多 Agent 协调仍为研究预览。

## 核心洞察

> "Harness 会编码对模型能力的假设，但模型在快速进化。脑手分离让两者独立演进。"

## 相关页面

- [[claude-code-harness]] — Harness 架构
- [[openclaw]] — OpenClaw 对比
- [[agent-architecture-patterns]] — Agent 架构
