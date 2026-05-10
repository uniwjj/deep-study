---
title: Claude Code Agent 循环
description: Claude Code 唯一的 1730 行 async generator 查询循环——10 个终止状态、7 个继续状态、4 层压缩管道、不可变状态转换模式
aliases: [Agent Loop, query loop, 查询循环, query.ts]
tags: [ai-agent, architecture, concept]
sources: [2026-05-10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code Agent 循环

`query.ts`（1,730 行）是一个单一的 `async function* agentLoop(...)`，是 Claude Code 的心脏。**每一次**交互——REPL、SDK、子 agent、headless——都通过它。

## 为什么用 AsyncGenerator

1. **背压**：generator 仅在消费者调用 `.next()` 时 yield。无缓冲区溢出。
2. **返回值语义**：`Terminal` 判别联合类型，10 个状态。调用者将终止原因作为类型化返回值获得。
3. **可组合性（`yield*`）**：外层 `query()` 通过 `yield*` 委托给 `queryLoop()`。
4. **惰性**：`function*` 体在第一次 `.next()` 前不执行——此时 React 已挂载。

## 循环状态对象（10 个字段）

| 字段 | 用途 |
|------|------|
| `messages` | 对话历史，每次迭代增长 |
| `toolUseContext` | 可变上下文：工具、abort 控制器、agent 状态 |
| `autoCompactTracking` | 压缩状态：回合计数器、连续失败次数 |
| `maxOutputTokensRecoveryCount` | 多回合恢复尝试次数（最大 3） |
| `hasAttemptedReactiveCompact` | 一次性守卫，防止无限反应式压缩循环 |
| `maxOutputTokensOverride` | 升级时设为 64K，之后清除 |
| `pendingToolUseSummary` | 上一迭代的 Haiku 摘要 Promise |
| `stopHookActive` | 防止阻止重试后重新运行 stop hooks |
| `turnCount` | 单调计数器，对 `maxTurns` 检查 |
| `transition` | 上一迭代为何继续（`Continue` 联合类型） |

## 不可变状态转换模式

每次继续点都构建**完整的新 State 对象**：

```javascript
const next: State = {
  messages: [...messagesForQuery, ...assistantMessages, ...toolResults],
  toolUseContext: toolUseContextWithQueryTracking,
  transition: { reason: 'next_turn' },
  // ... 所有 10 个字段显式设置
}
state = next
```

不是 `state.messages = newMessages`。每个继续点完全重建——自文档化，可读可知。

## 4 层上下文压缩管道

| 层 | 名称 | 机制 | 开关 |
|----|------|------|------|
| 0 | 工具结果预算 | 每消息大小限制 | 始终激活 |
| 1 | Snip Compact | 物理删除旧消息 | `HISTORY_SNIP` |
| 2 | Microcompact | 删除不再需要的工具结果 | `CACHED_MICROCOMPACT` |
| 3 | Context Collapse | 替换对话段为摘要 | `CONTEXT_COLLAPSE` |
| 4 | Auto-Compact | fork 对话进行摘要 | 默认启用 |

**Auto-Compact 熔断器**：3 次连续失败停止尝试。防止生产环境中曾观测到的无限 compact-fail-retry 循环（每天烧掉 250K API 调用）。

自动压缩阈值：
```
effectiveContextWindow = contextWindow - min(modelMaxOutput, 20000)
触发点 = effectiveContextWindow - 13,000 (AUTOCOMPACT_BUFFER_TOKENS)
```

## 模型流式处理

- `callModel()` 在 `while(attemptWithFallback)` 循环内启用模型回退
- `StreamingToolExecutor` 在流式处理期间启动工具
- **保留模式**：可恢复错误不 yield 给消费端。仅在**所有**恢复失败后才浮现。

## 错误恢复：升级阶梯

多重守卫防止无限循环：

- `hasAttemptedReactiveCompact` — 每种错误类型一次性
- `MAX_OUTPUT_TOKENS_RECOVERY_LIMIT = 3` — 多回合恢复硬上限
- 熔断器：auto-compact 3 次连续失败
- 错误响应上不运行 stop hooks — 防止 `error → hook blocking → retry → error` 循环

## 10 个终止状态

| 原因 | 触发 |
|------|------|
| `blocking_limit` | Token 到达硬限制，auto-compact OFF |
| `completed` | 正常完成（无工具调用或预算耗尽） |
| `aborted_streaming` | 模型流式期间用户中断 |
| `aborted_tools` | 工具执行期间用户中断 |
| `max_turns` | 达到 `maxTurns` 限制 |
| `stop_hook_prevented` | Stop hook 显式阻止 |
| `hook_stopped` | PreToolUse hook 阻止继续 |
| `prompt_too_long` | 所有恢复耗尽后 413 被保留 |
| `model_error` | 不可恢复的 API/模型异常 |
| `image_error` | 图像媒体错误 |

## 7 个继续状态

| 原因 | 触发 |
|------|------|
| `next_turn` | 正常工具调用继续 |
| `collapse_drain_retry` | Context collapse 排出了暂存 |
| `reactive_compact_retry` | 反应式 compact 成功 |
| `max_output_tokens_escalate` | 8K cap 命中，升级到 64K |
| `max_output_tokens_recovery` | 64K 仍被命中 |
| `stop_hook_blocking` | Stop hook 返回阻塞错误 |
| `token_budget_continuation` | Token 预算未耗尽 |

## 相关页面

- [[claude-code-architecture]] — 六抽象整体架构
- [[claude-code-tool-system]] — 14 步工具执行管道
- [[claude-code-concurrent-tools]] — 并发工具执行
- [[claude-code-api-layer]] — API 层（流式处理和 prompt cache）
- [[claude-code-context-compaction]] — 上下文压缩详解
