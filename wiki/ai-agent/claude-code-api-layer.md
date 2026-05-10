---
title: Claude Code API 层
description: Claude Code 的 API 通信层——四云提供商透明路由、字节级 prompt cache 感知系统提示构建、流式处理和错误恢复
aliases: [API Layer, 模型调用, queryModel, prompt cache, BetaMessageStream]
tags: [ai-agent, architecture, practice]
sources: [2026/05/10/Claude Code from Source.md, 2026/05/10/Claude-Code-Source-Analysis.pdf]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code API 层

API 层通过单一透明接口路由 4 个云提供商，构建具有字节级缓存感知的系统提示（一个放错位置的小节能破坏 50K+ token 的缓存），并维护会话稳定的不变量使 feature flag 变更不会造成隐形性能悬崖。

## 多云提供商客户端工厂

`getAnthropicClient()` 单一工厂函数：

- 4 个提供商：Direct API、Bedrock、Vertex AI、Foundry
- 分发完全由环境变量驱动，固定优先级
- 所有 4 个提供商特定 SDK 类通过 `as unknown as Anthropic` 类型转换
- 源码注释："we have always been lying about the return type"
- 每个提供商 SDK 动态导入——不使用的提供商永不加载
- 查询循环从不检查哪个提供商处于活动状态

## 系统提示构建

提示以**字符串段的数组**形式构建，存在关键的动态分界线：

```
分界线前：跨会话/用户/组织相同 → 最高层服务器端缓存（全局共享）
分界线后：用户特定 → 每会话缓存
```

**`systemPromptSection`**（安全，可缓存）vs **`DANGEROUS_uncachedSystemPromptSection`**（破坏缓存，需要 `_reason` 字符串参数）。

命名规范使违反在 code review 中可见，强制文档化原因，创造向安全默认值的心理摩擦。

### 2^N 问题

分界线前的每个布尔条件将全局缓存条目数**翻倍**。3 个条件 = 8 种变体，5 个 = 32 种。

源码中的注释：“Each conditional here is a runtime bit that would otherwise multiply the Blake2b prefix hash variants (2^N).”

规则：
- 编译时 feature flag 可以放在分界线前 ✓
- 运行时检查（是否 Haiku？auto 模式？）必须放在分界线后 ✗

## 三级 Prompt Cache

| 层级 | TTL | 范围 | 适用 |
|------|-----|------|------|
| 临时（默认） | ~5 分钟 | 每会话 | 所有用户 |
| 1 小时 | 60 分钟 | 订阅状态控制 | 订阅用户 |
| 全局 | 跨会话/组织 | 所有用户共享系统提示前缀 | 无 MCP 工具时 |

存在 MCP 工具时禁用全局缓存（MCP 工具定义因用户而异，会碎片化为数百万个唯一前缀）。

## Output Token Cap（槽位预留）

**默认上限：8,000 tokens**——不是 32K 或 64K。

- 生产数据 p99 输出为 4,911 tokens——标准限制过度预留 8-16 倍
- 命中上限时（<1% 请求），获得一次干净的 64K 重试
- 节省舰队规模下的显著成本

## 流式处理

使用**原始 `Stream<BetaRawMessageStreamEvent>`** 而非 SDK 的 `BetaMessageStream`。原因：`BetaMessageStream` 对每个 `input_json_delta` 事件调用 `partialParse()`——大工具调用 JSON 为 O(n²)。Claude Code 自己处理工具输入累积。

### 空闲看门狗

- 每收到一个 chunk 重置 `setTimeout`
- 90 秒无 chunk → 中止流式并回退到非流式重试
- 45 秒时发出警告

### 非流式回退

流式处理中途失败时回退到同步 `messages.create()`。处理代理故障（HTTP 200 但非 SSE body）。当流式工具执行活跃时禁用（会重复执行工具）。

## 错误处理与重试

`withRetry()` 是一个 async generator，产生用于 UI 显示的 `SystemAPIErrorMessage` 事件：

| 错误码 | 策略 |
|--------|------|
| 529（过载） | 等待 + 重试，可选降级快速模式 |
| 模型回退 | 主模型失败，尝试回退（Opus → Sonnet） |
| Thinking 降级 | 上下文窗口溢出 → 减少 thinking 预算 |
| OAuth 401 | 刷新 token，重试一次 |

## 设计模式

- **`DANGEROUS_` 命名规范**：使违反可见、强制文档化
- **基于 yield 的重试**：重试 wrapper 是 async generator，重试进度出现在事件流中
- **将稳定部分放在前、变化部分放在后**：最大化缓存命中率

## System Prompt 工程细节

### 5 级覆盖优先级

| 优先级 | 来源 | 用途 |
|--------|------|------|
| 1（最高） | `overrideSystemPrompt` | 测试/调试，完全替换 |
| 2 | Coordinator Mode | 多 Agent 协调 |
| 3 | Agent System Prompt | 领域特定 agent 指令 |
| 4 | Custom System Prompt | 用户 `--system-prompt` flag |
| 5（最低） | Default System Prompt | 标准 Claude Code 指令 |

`appendSystemPrompt` 始终追加到末尾，无视优先级。

### 3 种运行模式

| 模式 | 激活 | Prompt 特征 |
|------|------|-----------|
| **Simple** | `CLAUDE_CODE_SIMPLE=1` | 仅 3 行（identity/CWD/date），嵌入式/子进程用 |
| **Standard** | 默认 | 8 静态段 + N 动态段，数千 token |
| **Proactive (KAIROS)** | feature flag | 身份切换为"autonomous agent"，约束大幅简化 |

Proactive 模式下身份从 "help users with programming" 翻转为 **"You are an autonomous agent. Use tools to do useful work"** — 本质是不同的产品。

### Cache TTL 稳定性

`getPromptCache1hEligible()` 在**会话开始时锁定**。中途升级（free→paid）导致 TTL 从 5min→1hr，改变缓存 key → 缓存在会话中失效，**浪费约 20K token 的重复计算**。设计哲学：稳定性 > 准确性，偶尔用错 TTL 比中途缓存失效更便宜。

### Capybara v8 幻觉率

- Capybara v8 虚假声明率：**29-30%**
- Capybara v4：**16.7%**
- 对策：内部 prompt 注入 "capy v8 thoroughness counterweight" 指令
- 来源注释：`"False-claims mitigation for Capybara v8 (29-30% FC rate vs v4's 16.7%)"`

### v2.1.76 Cache 回归 Bug

两个原因叠加：
1. Attestation 数据每次请求变化 → cache hash 失效
2. 反蒸馏假工具定义每次不同 → hash 再次变化

正常 Turn 4 成本降至 Turn 1 的 13%（缓存命中），Bug 下膨胀到 $0.04→$0.40。根因：缓存前缀任何微小变化都会导致全量失效。

## 相关页面

- [[claude-code-architecture]] — 六抽象整体架构
- [[claude-code-agent-loop]] — 查询循环（API 调用者）
- [[claude-code-state-architecture]] — 粘性锁存器（缓存稳定性）
- [[claude-code-fork-subagent]] — Fork Subagent 的 prompt cache 复用
