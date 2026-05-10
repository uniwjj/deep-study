---
title: Claude Code 内置 Agent 详解
description: Claude Code 5 个内置 Agent 的完整系统提示词与设计哲学——Explore/Plan/Verification/General Purpose/Guide
aliases: [Builtin Agents, 内置Agent, Explore Agent, Plan Agent, Verification Agent, Guide Agent]
tags: [ai-agent, architecture, concept]
sources: [2026/05/10/Claude Code 技术解析.html]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 内置 Agent 详解

Claude Code 有 5 个精心设计的内置 Agent 类型，每个都有特化的 system prompt、工具集、模型选择和设计哲学。

## 1. Explore Agent（文件搜索专家）

| 属性 | 值 |
|------|-----|
| 模型 | **Haiku**（外部）/ inherit（内部） |
| 工具 | 只读（禁 Edit/Write/AgentTool） |
| CLAUDE.md | **省略**（omitClaudeMd: true） |
| 调用量 | 全舰队周均 **3400 万次** |

**设计哲学**：速度第一、纯读不写。

```
You are a file search specialist for Claude Code.
...READ-ONLY MODE - NO FILE MODIFICATIONS...
```

核心设计：
- Haiku 模型：速度快、成本低，纯读任务不需要强力推理
- 省略 CLAUDE.md：项目规范对文件搜索无意义，省 token
- 提示词要求并行：*"spawn multiple parallel tool calls"*
- 三档彻底度：`quick` / `medium` / `very thorough`

## 2. Plan Agent（架构设计师）

| 属性 | 值 |
|------|-----|
| 模型 | **inherit**（主 Agent 同款） |
| 工具 | 只读（同 Explore） |
| CLAUDE.md | 省略 |
| 职责 | 架构设计、出实现计划 |

**4 步流程**：Understand Requirements → Explore Thoroughly → Design Solution → Detail the Plan

强制结构化输出：必须以 `### Critical Files for Implementation` 结尾，列出 3-5 个关键文件路径。

**Plan Agent vs Plan Mode 对比**：

| 维度 | Plan Agent | Plan Mode |
|------|-----------|-----------|
| 本质 | 独立子 Agent，完整推理 | 主 Agent 权限限制模式 |
| 模型 | inherit（强模型） | 同主 Agent |
| 适用 | 复杂架构设计 | 简单操作前快速规划 |
| 切换 | 系统自动调用 | Shift+Tab 切换 |

Plan Agent 用强模型做深度架构分析，Plan Mode 只是限制主 Agent 不能写——本质不同。

## 3. Verification Agent（对抗性验证者）

| 属性 | 值 |
|------|-----|
| 模型 | inherit |
| 工具 | 只读 + /tmp 写 |
| 执行 | **红色后台**运行 |
| 输出 | PASS / FAIL / PARTIAL |

### 核心哲学：对抗性验证

```
Your job is not to confirm the implementation works — it's to try to break it.
```

**两大失败模式**：
1. **Verification avoidance**：找理由不运行测试，读代码然后写 "PASS"
2. **被前 80% 诱惑**：看到精美 UI 或通过的测试就给 PASS，忽略 20% 真正的问题

### 按变更类型的验证策略

| 变更 | 策略 |
|------|------|
| Frontend | 启动 dev server → Playwright 点击/截图/读 console |
| Backend/API | 启动服务 → curl endpoints → 验证响应形状 → 测试边界 |
| Bug Fix | **先重现原始 bug** → 验证修复 → 回归测试 |
| Refactoring | 现有测试 MUST pass → 对比公开 API 表面 |

### 认知陷阱自检清单

```
- "The code looks correct based on my reading" — 阅读不是验证。运行它。
- "The implementer's tests already pass" — 实现者也是 LLM。独立验证。
- "This is probably fine" — "可能"不是已验证。运行它。
- "I don't have a browser" — 检查过 mcp__playwright__* 了吗？
```

### 强制输出格式

```
### Check: POST /api/register rejects short password
**Command run:** curl -s -X POST ...
**Output observed:** {"error": "password must be at least 8 characters"}
**Expected vs Actual:** Expected 400. Got exactly that.
**Result: PASS**

---
VERDICT: PASS
```

每个 check 必须有真实命令执行证据——没有命令输出 = SKIP，不是 PASS。

## 4. General Purpose Agent（通用全能）

| 属性 | 值 |
|------|-----|
| 模型 | `getDefaultSubagentModel()`（动态） |
| 工具 | **全部**（`['*']`） |
| CLAUDE.md | 不省略 |
| 定位 | 兜底 Agent |

唯一的全权限内置 Agent：可读、写、编辑、运行任意 Bash、派生子 Agent。当需要跨领域能力时默认使用。

> "Complete the task fully — don't gold-plate, but don't leave it half-done."

模型由 `getDefaultSubagentModel()` 动态决定——唯一走实验性模型选择逻辑的内置 Agent。

## 5. Guide Agent（上下文感知向导）

| 属性 | 值 |
|------|-----|
| 模型 | **Haiku** |
| 工具 | 搜索 + 网络（无编辑类） |
| 权限 | **dontAsk** |
| 个性化 | 动态注入用户 skills/MCP/plugins/settings.json |

### 上下文感知个性化

```
## User's Current Configuration
**Available custom skills:** /commit, /review
**Available custom agents:** security-reviewer
**Configured MCP servers:** github-mcp, semgrep-mcp
**User's settings.json:** {"permissionMode": "bypassPermissions", "language": "zh-CN"}
```

Guide Agent 的 `getSystemPrompt()` 接收 `toolUseContext`，动态注入用户的全部运行时配置——skills、agents、MCP 服务器、settings.json。这使得 Guide Agent 能根据用户实际配置给出个性化建议。

### 会话复用

调用前检查是否有运行中的 `claude-code-guide` Agent，若有则用 **SendMessage** 继续对话而非新建——保持对话连续性。

### 为什么用 dontAsk

只回答问题、推荐工具，不修改文件，安全性有保障。dontAsk 消除不必要的确认弹窗，交互流畅。

## 五 Agent 对比总览

| Agent | 模型 | 工具权限 | CLAUDE.md | 核心价值 |
|-------|------|---------|-----------|---------|
| Explore | Haiku | 只读 | 省略 | 极速搜索 |
| Plan | inherit | 只读 | 省略 | 架构设计 |
| Verification | inherit | 只读+tmp写 | — | 对抗性验证 |
| General Purpose | 动态 | 全部 | 保留 | 兜底全能 |
| Guide | Haiku | 搜索+网络 | — | 个性化帮助 |

## 相关页面

- [[claude-code-subagent]] — 子代理委托架构（AgentTool 决策树/runAgent 生命周期）
- [[claude-code-multi-agent-mechanism]] — 三种多 Agent 机制总览
- [[claude-code-internal-mechanisms]] — Verification Agent 在内部的使用
