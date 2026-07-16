---
title: Agent 架构模式
description: AI Agent 系统的 10 大设计模式——从 AsyncGenerator 循环到流式推测执行，Claude Code 源码提炼的通用架构原则
aliases: [agent-architecture-patterns, Agent Architecture Patterns, 架构模式, 设计模式]
tags: [ai-agent, concept, architecture]
sources: [2026/05/10/Claude-Code-Source-Analysis.pdf, 2026/07/16/Agent 治理：用 Hook 堵住 LLM 的偷懒、越权与失忆.html]
created: 2026-05-10
updated: 2026-07-16
---

# Agent 架构模式

从 Claude Code ~2000 文件 TypeScript 单体中提炼的 10 大通用设计模式，适用于任何 Agent 系统。

## 10 大设计模式

| # | 模式 | 核心思想 | 为什么重要 |
|---|------|---------|-----------|
| 1 | **AsyncGenerator Agent 循环** | 主循环是 `while` + async generator，并非回调 | 支持暂停/恢复、流式输出、优雅中断 |
| 2 | **自描述工具接口** | 工具自己声明名称、描述、参数 schema | 模型自动理解工具能力，无需中央编排 |
| 3 | **访问模式分离状态** | 高频读（缓存）+ 低频写（持久化），两层状态架构 | 避免每次循环都 IO，性能优化 |
| 4 | **权限模式级联** | 父 bypass 胜出 → 后台禁弹窗 → 子 Agent 会话规则替换 | 多 Agent 安全边界清晰 |
| 5 | **递归 Agent** | Agent 可以生成新 Agent，AgentTool 创建隔离上下文 | 任务自然分解，无需预定义层级 |
| 6 | **漏斗式启动** | 从最小上下文开始，按需扩展 | 避免启动时就加载所有 CLAUDE.md |
| 7 | **粘性缓存锁** | Prompt cache 锁定高频前缀 | token 成本降至 10% |
| 8 | **DANGEROUS 命名** | 高风险工具名显式标注 | 开发者一眼识别安全边界 |
| 9 | **流式推测执行** | 模型还在输出时就开始执行工具调用 | 延迟掩藏，体验零等待 |
| 10 | **安全分区并发批处理** | Read/Grep/Glob 始终并发安全 | 工具调用无副作用冲突 |

## 架构原则

- **生成器循环优于回调**：状态管理更自然，错误恢复更简单
- **文件存储优于数据库**：Agent 状态以文件为原子单位，便于检查点和恢复
- **工具自描述优于中央编排**：新增工具无需改动调度器
- **Hook 优于插件**：生命周期拦截比组件加载更灵活。Hook 切面（beforeTool/afterTool/beforeModel/afterModel）可在不改业务循环、不动工具实现的前提下，挂载长文本 offload、HITL 门禁、上下文联动等横切逻辑。DECO 的实践在一套 Hook 链上挂了十余个 Hook，参见 [[agent-hook-governance]]

## 核心架构比例

Claude Code 中：98.4% 代码是操作基础设施，只有 1.6% 是 Agent 循环本身。这说明：**Agent 架构的本质不是循环逻辑，而是循环周围的工具、状态、权限、记忆系统。**

## 相关页面

- [[claude-code-architecture]] — Claude Code 架构全景与核心文件速查
- [[claude-code-agent-loop]] — Agent 主循环的源码实现
- [[agent-vs-framework-vs-harness]] — Agent vs Framework vs Harness 三层区分
- [[dive-into-claude-code]] — 学术论文：13 条设计原则
- [[agent-harness-overview]] — Harness 六承重层
- [[agent-hook-governance]] — Hook 护栏治理：DECO 的框架层 Hook 实践（offload/HITL/上下文联动）
