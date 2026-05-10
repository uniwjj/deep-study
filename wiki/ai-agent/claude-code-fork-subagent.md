---
title: Claude Code Fork Subagent
description: Fork Subagent 机制——三层冻结确保字节级缓存复用、递归 fork 双重防护、sync-to-async 转换、4 样继承与 10% 成本
aliases: [Fork Subagent, 缓存复用, prompt cache, CacheSafeParams]
tags: [ai-agent, concept, architecture]
sources: [2026/05/10/Claude Code 多Agent实现机制.md, 2026/05/10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code Fork Subagent

Fork Subagent 是 Claude Code 中一个**隐蔽但精妙**的机制——通过复用 Anthropic API 的 prompt 缓存，将 subagent 的**成本和延迟大幅降低**。

## 问题背景

Claude Code 的 system prompt 长度是**上万 token**，包含大量工具说明、规范约定、用户上下文。每派一个常规 subagent，API 端对这一万多 token 重新从头计算，导致：
- **成本**：input token 重新计费
- **延迟**：首 token 等待时间更长
- 子代理派得越频繁，开销线性放大

Anthropic 提供了 **prompt 缓存机制**：如果 API 请求前缀与之前某次请求**字节级完全一致**，此前缀可直接走缓存，价格降至 10%，延迟大幅降低。

## Fork Subagent 核心思路

派一个子 agent 出去干活，但其 API 请求前缀与父 agent **字节级完全相同**，让 Anthropic 的缓存命中。

## 五样必须对齐的内容

`CacheSafeParams` 类型定义了五项必须与父 agent 完全一致的内容：

| 对齐项 | 说明 |
|--------|------|
| `systemPrompt` | 系统提示，对齐第一位 |
| `userContext` | 用户上下文（CLAUDE.md 等动态内容） |
| `systemContext` | 拼接在 system prompt 后的环境信息 |
| `toolUseContext` | 工具池、模型等上下文 |
| `forkContextMessages` | 父 agent 的消息前缀，决定分叉点 |

任意一样有**一个字节**不一致，缓存立即失效。

## 三层冻结机制

### Layer 1：通过 threading 冻结 system prompt

为什么不重新调用 `getSystemPrompt()`？因为 **GrowthBook feature flag 的内存状态会变化**——父 agent 第一轮调用时 flag 返回 false，到 fork child 启动时可能变成 true。重新生成得到的字节不同 → **缓存立即失效**。

已渲染的 system prompt 存于 `toolUseContext.renderedSystemPrompt`，fork child 直接收到这个精确字符串。

### Layer 2：通过 Exact Passthrough 冻结工具定义

正常子 agent 走 `resolveAgentTools()`。Fork 设 `useExactTools: true`，完全跳过过滤。子 agent 获得父 agent 完整工具池——甚至包括 Agent 工具（尽管无法直接使用）。

### Layer 3：消息数组构造

`buildForkedMessages()` 构造最后两条消息：
- 克隆的 assistant 消息（所有 fork child 相同，使用**占位符工具结果**）
- user 消息（封装每个 child 的不同指令）

占位符工具结果是“为了简洁和统一而选择的一个谎言”——不是真正的工具输出，但保证 byte-identical 前缀。

## Fork Boilerplate XML 标签

每个 child 的指令包装在 boilerplate XML 中，约 10 条规则：
- 覆盖父的 fork 指令："You ARE the fork. Do NOT spawn sub-agents."
- 静默执行，一次性报告。工具调用间不输出对话文本
- 不超出指令范围
- 结构化输出格式：Scope / Result / Key files / Files changed / Issues

## 递归 Fork 双重防护

1. **主守卫**：`querySource` 检查。Fork child 有 `querySource: 'agent:builtin:fork'`。`call()` 在允许 fork 路径前检查
2. **后备守卫**：消息扫描。在对话历史中扫描 boilerplate XML 标签。双重防护（belt and suspenders）

## Sync-to-Async 转换

Fork child 在前台启动。超过一定时间可转为后台。

机制：前台 iterator 在 "next message" 与 "background signal" 之间竞争（`Promise.race`）。触发 background signal → iterator 优雅终止 → 新 `runAgent()` 实例以 `isAsync: true` 参数生成——但使用**相同 agent ID 和消息历史**，保证连续性。

## 经济学

具象数字：
- Child 1：48,700 tokens @ 全价（缓存 miss）
- Child 2-5：48,500 tokens @ 10% 价格（缓存命中）+ ~200 tokens @ 全价（差异部分）
- Child 2-5 实际成本：~5,050 tokens 等价

对于有 100K tokens 历史并生成 8 个并行 fork 的会话，缓存节省可超 90%。

## 三项设计张力

1. **隔离 vs 缓存效率**：fork child 继承所有内容，包括无关历史
2. **安全性 vs 缓存效率**：Agent 工具留在 child 工具池（尽管无法使用），移除会破坏缓存
3. **简洁性 vs 缓存效率**：占位符工具结果是“一个谎言”，选择它是为了简洁和统一

## 关键实现细节

### 1. System prompt 不重新生成

Fork Subagent 的合成定义中，`getSystemPrompt` 返回空字符串：

```typescript
// src/tools/AgentTool/forkSubagent.ts
export const FORK_AGENT = {
  getSystemPrompt: () => '',  // 返回空串！
  tools: ['*'],               // 用父的完整工具池
  model: 'inherit',           // 继承父的模型
  permissionMode: 'bubble',   // 权限弹窗浮到父终端
}
```

这不是偷懒，而是**精心设计**：Fork subagent 的 system prompt 根本不走生成函数，而是直接用父 agent 已经渲染好的那份字节。如果重新生成，某些动态字段可能变化（功能开关缓存状态等），导致差一个字符，缓存就没了。

### 2. 与 Coordinator 模式互斥

Fork 机制和 Coordinator 模式职责重叠，开启 Coordinator 时自动禁用 Fork：

```typescript
export function isForkSubagentEnabled(): boolean {
  if (feature('FORK_SUBAGENT')) {
    if (isCoordinatorMode()) return false  // 互斥！
    return true
  }
  return false
}
```

## 适用场景 vs 常规 Subagent

| | Fork Subagent | 常规 Subagent |
|--|-------------|--------------|
| 适用 | 需要父完整上下文 + 不污染主循环 | 明确专业分工 |
| 举例 | Ctrl+F 生成 PR 描述、`/btw` 命令 | Explore 搜代码、Plan 做规划 |
| System prompt | 复用父的字节原样 | 独立定制 |
| 成本 | ~10% | 100% |

## 工程启示

**成本优化本身就是能力的一部分**。Fork 机制将 subagent 成本降至约 10%，意味着可以更频繁地派 subagent，反过来扩大了整个 agent 系统的能力边界。

对于自建 agent 系统的核心教训：设计 subagent 时，考虑它的 prompt 前缀能否复用父 agent 的缓存，能省 80-90% 的成本。

## 相关页面

- [[claude-code-multi-agent-mechanism]] — 多 Agent 机制总览
- [[claude-code-subagent]] — 常规子代理架构
- [[claude-code-coordinator-mode]] — Coordinator 并行协作模式
- [[dive-into-claude-code]] — 学术论文分析
- [[claude-code-api-layer]] — API 层（prompt cache 三层体系）
- [[claude-code-state-architecture]] — 粘性锁存器（缓存稳定的关键）
