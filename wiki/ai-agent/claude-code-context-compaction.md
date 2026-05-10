---
title: Claude Code 上下文压缩
description: Claude Code 的5层渐进压缩管道——预算削减、裁剪、微压缩、上下文折叠、自动压缩，以及 CLAUDE.md 四级记忆体系
aliases: [上下文压缩, context compaction, compaction pipeline]
tags: [ai-agent, concept, architecture]
sources: [2026/05/10/Dive into Claude Code.txt, 2026-05-10/Claude-Code-Source-Analysis.pdf]
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

## 9 段式结构化压缩模板

| # | 段 | 内容 | 优先级 |
|---|----|------|--------|
| 1 | Primary Request and Intent | 用户所有明确请求和意图 | 最高 |
| 2 | Key Technical Concepts | 讨论的技术、框架 | 高 |
| 3 | Files and Code Sections | 文件名、代码片段、修改记录 | 高 |
| 4 | Errors and Fixes | 错误及修复 | 中高 |
| 5 | Problem Solving | 问题解决过程 | 中 |
| 6 | **All User Messages** | 所有非工具结果的用户消息 | **最高** |
| 7 | Pending Tasks | 明确要求但未完成的任务 | 高 |
| 8 | **Current Work** | 压缩前进行中的工作（需逐字引用） | **最高** |
| 9 | Optional Next Step | 下一步建议 | 中 |

**第 6 段是关键设计决策**：列出所有非工具结果的用户消息。用户消息同时编码信息和约束——丢失一句"别用 var"就会导致回退。

## Full vs Partial Compact

| 维度 | Full | Partial |
|------|------|---------|
| 范围 | 全部对话历史 | 仅部分，其余保留原文 |
| 触发 | autoCompact、手动 /compact | 智能分割点选择 |
| Prompt cache | 完全失效 | `direction='from'` 保留前半缓存 |
| 信息丢失 | 较大 | 较小 |

两种方向：
- `direction='from'`：保留旧消息原文，压缩新消息。旧缓存完全命中
- `direction='up_to'`：压缩旧消息，保留新消息原文。缓存失效但新上下文完整

## 两阶段压缩

**Phase 1 (analysis)**：模型在 `<analysis>` 标签中自由思考确保全面覆盖，不计入最终输出。

**Phase 2 (summary)**：基于分析输出 9 段式结构化摘要，成为下一轮对话的初始上下文。直接要求结构化输出会导致遗漏细节。

## 防漂移设计：逐字引用

第 8-9 段要求**精确逐字引用**最新对话内容——不是摘要，不是描述：
> "This should be verbatim to ensure there's no drift in task interpretation."

每次压缩都是长对话→短摘要的"翻译"，翻译会丢失语义细节。"Make the button smaller" 压缩为 "adjust button styling" 会导致模型还改颜色。逐字引用将模型锚定到原始用户意图，防止多次压缩累积偏移。

## Sonnet 4.6 压缩失败率

- Sonnet 4.6：**2.79%** 失败率
- Sonnet 4.5：**0.01%** 失败率
- 失败模式：模型在压缩期间尝试工具调用（尽管有 "no tools" 指令）。maxTurns: 1 下，工具调用被拒绝 = 无文本输出 = 压缩失败
- 缓解：添加激进的 "no-tools" 前置声明。模型版本变化可破坏现有 prompt 策略

## 信息丢失风险矩阵

| 信息类型 | 丢失风险 | 原因 |
|---------|---------|------|
| 隐式偏好 | **高** | "别用 var" 等一句话极易被压缩掉 |
| 试过但失败的方法 | **高** | 错误保留但方法细节易丢失 |
| 中间讨论 | **高** | 讨论 A,B,C 选 B → 压缩为 "chose B" |
| 早期代码修改 | **中** | 长会话中早期修改被后续覆盖 |
| 用户直接请求 | **低** | Section 6 要求完整保留 |
| 当前进行中工作 | **低** | Section 8 要求逐字引用 |

## 相关页面

- [[dive-into-claude-code]] — 论文总览
- [[claude-code-permission-system]] — 权限系统
- [[claude-code-extensibility]] — 可扩展性机制（CLAUDE.md 是其一部分）
- [[claude-code-session-persistence]] — 会话持久化（压缩与持久化紧密相关）
- [[claude-code-subagent]] — 子代理（摘要返回也是上下文管理策略）
