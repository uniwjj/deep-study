---
title: Claude Code 内部机制
description: Claude Code 源码中的内部机制——Undercover Mode 泄露防护、反蒸馏防御、Regex 情绪检测、Verification Agent 对抗性验证
aliases: [Internal Mechanisms, 内部机制, Undercover Mode, 反蒸馏, Sentiment Detection]
tags: [ai-agent, architecture, concept]
sources: [2026/05/10/Claude-Code-Source-Analysis.pdf]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 内部机制

Claude Code 源码中有一套内部模型保护机制，通过编译时常量 `USER_TYPE === 'ant'` 控制——外部构建物理上不含这些代码。

## Undercover Mode（身份伪装）

### 激活逻辑

仓库分类白名单 `INTERNAL_MODEL_REPOS`（~20 个仓库：claude-cli-internal、anthropic、apps、casino 等）：

| 分类 | 条件 |
|------|------|
| `internal` | remote 匹配白名单（私有仓库） |
| `external` | 有 remote，不在白名单（公开仓库） |
| `none` | 无 remote URL（非 git 仓库） |

**默认 ON**：只有明确确认的内部仓库才禁用。源码注释："This guards against model codename leaks — if we're not confident we're in an internal repo, we stay undercover."

### 生效行为

1. 从 system prompt 中剥离所有模型名称
2. 隐藏内部模型代号（Capybara、Tengu 等）
3. 隐藏未发布模型版本（opus-4-7）
4. 隐藏内部工具和 Slack 引用
5. 提交信息伪装为人类贡献者——描述代码改动，绝不提及 AI
6. 首次在公开仓库自动激活时弹出一次性通知

**没有强制关闭选项** ——安全优先于便利。

## 反蒸馏防御

三种技术，按成熟度排序：

### a) 假工具定义注入 API 请求

在 API 请求中注入**虚假的工具定义**。如果竞争对手拦截 API 请求用于训练自己的模型（蒸馏），这些被污染的定义会破坏训练数据集。社区对此存在伦理争议。

### b) `redact-thinking-2026-02-12` beta header

模型的推理链在 API 响应中被隐藏。即使对手拦截了 API 响应，也无法获取思维链——而这正是对蒸馏最有价值的数据。

### c) Slate Prism POC

服务端输出摘要 + 加密签名，确保下游消费者无法获取原始输出。泄露版本中仍为概念验证阶段。

## Sentiment Detection（情绪检测）

**用正则表达式做情绪检测，而非 LLM**。

| 指标 | 对比 |
|------|------|
| 速度 | 比最快 LLM 调用快 ~10,000 倍 |
| 成本 | 接近零 |
| 确定性 | 强，适合指标对比 |
| 依赖 | 无需额外 API 调用 |

**纯观测指标**：仅记录情绪信号，不改变 AI 行为。用于团队理解用户体验趋势，非面向用户的功能。

> 这呼应了搜索策略的主题：不是所有问题都需要最先进的技术。Regex 情绪检测精度低，但足够"粗略感知用户情绪趋势"。

## Verification Agent（对抗性验证）

**内部功能，A/B 测试中**。完成复杂实现任务后自动启动独立 agent 进行对抗性验证。

### 核心原则

> "Execution agent cannot grade their own homework."

执行 agent 自己的检查、caveat、fork 自检都不算数——只有验证者给出结论。

### 三级判定

| 判定 | 动作 |
|------|------|
| **PASS** | 任务完成 |
| **FAIL** | 必须修复并重新验证直到 PASS |
| **PARTIAL** | 存在问题，需关注 |

### 设计规则

1. Verification Agent 与 Execution Agent **完全独立**
2. FAIL 必须走 fix → re-verify 循环
3. 验证者对比命令输出与执行 agent 实际命令的重新运行结果
4. 本质是 AI 化的 code review——一个人写代码，另一个人审查，只是两者都是 AI

## 编译时分离：USER_TYPE

`USER_TYPE` 是**编译时 `--define`**，不是运行时环境变量。外部构建中 `process.env.USER_TYPE === 'ant'` 被常量折叠为 `false`，然后 Dead Code Elimination 移除整个 `false` 分支及所有依赖模块。结果：外部二进制**物理上不含**任何内部代码。

## 相关页面

- [[claude-code-hidden-features]] — 87 个 Feature Flag
- [[claude-code-architecture]] — 整体架构
- [[claude-code-subagent]] — 子代理委托
- [[claude-code-search-strategy]] — 搜索策略（同主题：简单工具 > 复杂工具）
