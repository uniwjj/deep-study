---
title: Claude Code vs OpenCode
description: 同模型不同 Agent 框架效果差异根源——上下文管理（三层压缩 vs 单层）、Prompt 工程、Agent 通信（SubAgent/Fork/Teammate）、工具编排四维度对比
aliases: [OpenCode, Claude Code对比, CC, Claude CLI]
tags: [ai-agent, comparison, tool]
sources: [2026/04/08/从源码角度聊聊 Claude Code 和 OpenCode 的效果区别.md]
created: 2026-05-09
updated: 2026-05-09
---

# Claude Code vs OpenCode

> "基础设施越精细，模型能力释放得越充分。"

## 四维度对比

| 维度 | Claude Code | OpenCode |
|------|-------------|----------|
| **上下文管理** | 三层递进压缩 | 单层（22 行代码） |
| **Prompt 工程** | 静态/动态分割+prompt cache | 6 套模板，无 cache |
| **Agent 通信** | SubAgent/Fork/Teammate 三种 | 仅 Task Tool |
| **工具编排** | 自动并行分区+2600 行安全验证 | Batch Tool 模式 |

## 上下文管理（差距最大）

### Claude Code 三层压缩
- **Micro Compact**：零 API 调用，纯 token 估算裁剪旧工具输出
- **Session Memory Compact**：提取结构化会话记忆，保留最近 10-40K tokens
- **Full Compact**：Fork 专用 agent 生成完整摘要，消耗 API

触发机制：8K 压缩缓冲区、20K 警告阈值、连续失败 3 次熔断。代码注释记录：曾发现 1279 个 session 连续失败 50+ 次，每天浪费 25 万次 API 调用。

### OpenCode
单层 overflow.ts（22 行），token 超阈值就触发，固定模板摘要。

## Prompt 工程

- Claude Code：`SYSTEM_PROMPT_DYNAMIC_BOUNDARY` 切分静/动态，静态走跨用户 cache。Fork agent 复用父进程提示字节缓冲区
- OpenCode：按模型 ID 分发 6 套模板，无 cache 优化

## Agent 通信

- Claude Code 三种隔离级别：SubAgent（完全孤立法）/ Fork（继承上下文+共享 cache）/ Teammate（独立进程+文件系统邮箱）
- OpenCode：仅 Task Tool，每次新建 session

## 工具编排

- Claude Code：`isConcurrencySafe()` 自动并行分区（最多 10 并发），写入串行。BashTool 2600 行验证器
- OpenCode：Batch Tool 显式包数组（最多 25 并发），无自动安全验证

## 结论

差距根源不在于模型能力，而在于工程投入。Claude Code 的工程集中在"看不见的基础设施"上。OpenCode 优势：75+ 模型提供商、完整 TUI、开源 120K+ star。

## 相关页面

- [[claude-code-architecture]] — Claude Code 源码架构
- [[claude-code-harness]] — Harness 架构
- [[agent-architecture-patterns]] — Agent 架构模式
