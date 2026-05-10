---
title: Claude Code 上下文压缩
description: Claude Code 的5层渐进压缩管道——预算削减、裁剪、微压缩、上下文折叠、自动压缩，以及 CLAUDE.md 四级记忆体系
aliases: [上下文压缩, context compaction, compaction pipeline]
tags: [ai-agent, concept, architecture]
sources: [2026/05/10/Dive into Claude Code.txt]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 上下文压缩

上下文窗口是 Claude Code 的**绑定资源约束**。每次模型调用前，执行一个**五层压缩管道**，体现"上下文作为稀缺资源"原则。

## 设计原则

采用**最轻破坏优先**的懒降级策略：
- 先尝试最廉价的压缩方法
- 仅在便宜策略不足时升级
- 五层互动可能产生用户难以完全预测的行为

## 五层压缩管道

在 `query.ts` 中每次模型调用前顺序执行：

### 1. 预算削减（始终激活）
`applyToolResultBudget()` — 对工具结果强制执行单消息大小限制。超大输出替换为内容引用。豁免工具的 `maxResultSizeChars` 不为有限值。

### 2. 裁剪（`HISTORY_SNIP` 开关控制）
`snipCompactIfNeeded()` — 轻量级修剪，移除较旧的历史段。返回 `{messages, tokensFreed, boundaryMessage}`。节省的 token 数需显式传递给自动压缩。

### 3. 微压缩（`CACHED_MICROCOMPACT` 开关控制）
细粒度压缩，始终运行基于时间的路径，可选运行缓存感知路径。边界消息推迟到 API 响应后，使用实际的 `cache_deleted_input_tokens`。

### 4. 上下文折叠（`CONTEXT_COLLAPSE` 开关控制）
**读时虚投影**：不修改存储历史，创建消息数组的投影视图。摘要消息存储在折叠存储而非 REPL 数组中。模型看到折叠版本，但完整历史仍可用于重建。

### 5. 自动压缩（默认启用，可禁用）
`compactConversation()` — 完整的模型生成摘要。仅在前面四个整形器执行后上下文仍超阈值时触发。执行 PreCompact 钩子，调用模型生成压缩摘要。

## 压缩后的消息构建

```
[边界标记, ...摘要消息, ...保留消息, ...附件, ...钩子结果]
```

边界标记记录 `headUuid`、`anchorUuid`、`tailUuid`，支持读时链修补。压缩从不修改或删除之前写入的转录行，只追加新的边界和摘要事件。

## 上下文之外的压缩策略

| 策略 | 说明 |
|------|------|
| CLAUDE.md 懒加载 | 基础层次在会话启动时加载，嵌套目录文件仅在 Agent 读取这些目录时加载 |
| 延迟工具模式 | ToolSearch 启用时，工具仅在初始上下文中包含名称，完整模式按需加载 |
| 子代理摘要返回 | 子代理仅返回摘要文本给父代理，不返回完整历史 |
| 单工具结果预算 | 单个工具结果上限可配置，防止单个冗长输出消耗不成比例的上下文 |

## CLAUDE.md 四级记忆体系

`getUserContext()` 加载四层指令文件（`claudemd.ts`）：

| 层级 | 路径 | 说明 |
|------|------|------|
| 1. 托管记忆 | `/etc/claude-code/CLAUDE.md` (Linux) | OS 级策略，所有用户 |
| 2. 用户记忆 | `~/.claude/CLAUDE.md` | 私有全局指令 |
| 3. 项目记忆 | `CLAUDE.md`, `.claude/CLAUDE.md`, `.claude/rules/*.md` | 签入代码库的指令 |
| 4. 本地记忆 | `CLAUDE.local.md` | gitignored，私有项目特定指令 |

文件按**优先级逆序**加载——后加载的文件获得更多模型关注。使用 `@include` 指令支持模块化指令集。

**重要架构选择**：CLAUDE.md 内容作为**用户上下文**（user message）传递而非系统提示，意味着模型遵循这些指令是**概率性而非保证性**的。权限规则（拒绝优先顺序）提供确定性执行层。

## 上下文窗口组装顺序

```
系统提示 → 环境信息(git状态) → CLAUDE.md层次 → 路径范围规则 →
自动记忆 → 工具元数据 → 对话历史 → 工具结果 → 压缩摘要
```

部分上下文源在窗口构建后**延迟注入**（相关记忆预取、MCP 指令增量、Agent 列表增量、后台 Agent 任务通知）。

## 透明度代价

压缩对用户**大体不可见**：
- 预算削减将长输出替换为引用
- 上下文折叠在无用户可见输出的情况下操作
- 微压缩的缓存感知行为增加了不透明度

## 相关页面

- [[dive-into-claude-code]] — 论文总览
- [[claude-code-permission-system]] — 权限系统
- [[claude-code-extensibility]] — 可扩展性机制（CLAUDE.md 是其一部分）
- [[claude-code-session-persistence]] — 会话持久化（压缩与持久化紧密相关）
- [[claude-code-subagent]] — 子代理（摘要返回也是上下文管理策略）
