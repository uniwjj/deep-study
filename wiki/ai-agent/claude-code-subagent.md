---
title: Claude Code 子代理委托
description: Claude Code 的子代理委托架构——AgentTool 10步决策树、runAgent() 15步生命周期、4层权限隔离、3种Abort行为
aliases: [子代理, subagent, AgentTool, 多代理]
tags: [ai-agent, concept, architecture]
sources: [2026/05/10/Dive into Claude Code.txt, 2026/05/10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 子代理委托

Claude Code 采用**父-子任务委托模式**，通过 `AgentTool` 工具将工作分派到独立上下文窗口的代理。

## 六种内置子代理类型

| 类型 | 模型 | 特点 |
|------|------|------|
| **Explore** | Haiku | 最极致优化：只读工具集合，移除 FileEdit/FileWrite/NotebookEdit/Agent，省略 CLAUDE.md 和 git 状态。全舰队每周 3400 万次调用 |
| **Plan** | inherit | 四步流程：理解需求 → 全面探索 → 设计方案 → 细节规划 |
| **Verification** | inherit | 对抗性测试者，始终后台运行，system prompt 明确列出模型可能找的借口 |
| **General-purpose** | — | 默认 agent，全工具访问，不省略 CLAUDE.md |
| **Claude Code Guide** | Haiku | 回答关于 Claude Code 本身的问题，`dontAsk` 权限模式 |
| **Statusline Setup** | Sonnet | 仅限制 Read 和 Edit 工具 |

## AgentTool call() 决策树（10 步路由）

在 `runAgent()` 被调用之前，`call()` 按以下顺序决策：

1. 是否为 teammate 生成？（同时设 `team_name` + `name`）→ `spawnTeammate()`
2. 从 `subagent_type` 解析实际 agent 类型
3. 是否为 fork 路径？（`effectiveType === undefined`）→ 递归 fork 守护检查
4. 从 `activeAgents` 列表解析 agent 定义
5. 检查必要 MCP 服务器（等待挂起的最长 30 秒）
6. 解析隔离模式（参数覆盖 agent 定义）
7. 确定同步 vs 异步
8. 组装 worker 工具池
9. 构建 system prompt 和 prompt 消息
10. 执行（异步 → `registerAsyncAgent`；同步 → 迭代 `runAgent`）

## runAgent() 15 步生命周期

`runAgent()`（~400 行）是驱动子 agent 完整生命周期的唯一函数：

**步骤 1**：模型解析 — 透传覆盖 > agent 定义 > 父模型 > 默认值

**步骤 2**：Agent ID 创建 — 模式 `agent-<hex>`，hex 来自 `crypto.randomUUID()`

**步骤 3**：上下文准备 — fork agent 克隆父对话历史，普通 agent 空白开始

**步骤 4**：CLAUDE.md 剥离 — Explore/Plan 设置 `omitClaudeMd: true`（对只读搜索无意义）

**步骤 5**：权限隔离（四层）：
1. 父在 bypassPermissions/acceptEdits/auto 模式时父模式胜出
2. 后台 agent 自动禁止弹窗
3. 可弹窗的后台 agent 设 `awaitAutomatedChecksBeforeDialog`
4. 提供 `allowedTools` 时整体替换 session 级允许规则

**步骤 6**：工具解析 — fork 用 `useExactTools: true` 直接透传；普通走 `resolveAgentTools()` 分层过滤

**步骤 7**：System Prompt — fork 接收父已渲染的字节（`override.systemPrompt`）；普通调用 `getAgentSystemPrompt()`

**步骤 8**：Abort 控制器隔离 — 三种行为：Override（恢复后台化 agent）、异步 agent（新独立 controller，Escape 只杀父）、同步 agent（共享父 controller）

**步骤 9-15**：
9. Hook 注册 — agent 定义声明 hooks，通过 `agentId` 限定生命周期
10. Skill 预加载 — frontmatter 指定 skills，以 user 消息前置到 agent 会话
11. MCP 初始化 — 叠加到父 agent 客户端之上（非替换）
12. 上下文创建 — `createSubagentContext()` 按隔离策略组装 `ToolUseContext`
13. Cache-Safe Params Callback — 供后台摘要使用，不干扰主对话
14. Query Loop — 复用 `query()`，每条消息记录到侧链转录
15. 清理 — finally 块清理 MCP 连接、hooks、缓存跟踪、文件状态、perfetto 追踪、todo 条目、孤儿 shell 进程

## Agent 定义来源（四个优先级）

1. 内置 agent — TypeScript 硬编码
2. 用户 agent — `.claude/agents/` 目录下 Markdown 文件
3. 插件 agent — 通过 `loadPluginAgents()` 加载
4. 策略 agent — 组织策略配置加载

## 五维度 Agent 设计模式语言

| 维度 | 决策问题 |
|------|---------|
| 能看到什么？ | 上下文不是免费的，每个 token 有成本和记忆占用 |
| 能做什么？ | 工具访问决定能力边界 |
| 如何决策？ | 模型选择和 system prompt 决定推理质量 |
| 如何报告？ | 输出格式决定父 agent 的综合能力 |
| 如何持久化？ | 生命周期管理决定资源效率 |

## 自定义 Agent 定义

通过 `.claude/agents/*.md` 文件（`loadAgentsDir.ts`）：

- Markdown 正文作为 Agent 的系统提示
- YAML frontmatter 配置：`description`, `tools` (allowlist), `disallowedTools`, `model`, `effort`, `permissionMode`, `mcpServers`, `hooks`, `maxTurns`, `skills`, `memory`, `background`, `isolation`

一个自定义 Agent 可以是**完全配置的隔离子系统**。

## 三种隔离模式

| 模式 | 行为 |
|------|------|
| **Worktree** | 创建临时 Git worktree，子代理有独立仓库副本 |
| **Remote**（内部） | 在远程 Claude Code Remote 环境启动，始终在后台运行 |
| **In-process**（默认） | 共享文件系统，但运行在独立对话上下文中 |

Worktree 隔离通过 Git 内置机制提供文件系统级分离，**零外部依赖性**。

## 权限覆盖逻辑

- 子代理定义了 `permissionMode` 时，除非父代理已在 `bypassPermissions`/`acceptEdits`/`auto` 模式
- 异步代理：通过 `canShowPermissionPrompts` → `bubble` 模式 → 默认（同步显示，异步不显示）级联决定
- `allowedTools` 提供时，采用两级权限作用域模型：SDK 级权限保留，会话级规则被替换

## 侧链转录

每个子代理写入独立的 `.jsonl` + `.meta.json` 文件（`sessionStorage.ts`, `runAgent.ts`）：

- **仅子代理的最终响应文本和元数据**返回父代理
- 完整子代理历史**永不进入**父上下文窗口
- 保护了"上下文作为瓶颈"原则

## Agent 团队协调

多实例协调使用**文件锁**（而非消息代理）：
- 任务从共享列表通过基于锁文件的互斥认领
- 零依赖部署 + 完全可调试（任何 Agent 状态通过纯文本 JSON 文件检查）
- 代价：吞吐量降低

## 与 SkillTool 的区别

| | SkillTool | AgentTool |
|---|----------|----------|
| 作用 | 将指令注入当前上下文窗口 | 生成新的隔离上下文窗口 |
| 上下文成本 | 低（仅描述） | 需要独立上下文 |
| 返回 | 直接执行结果 | 摘要文本 |

## 相关页面

- [[claude-code-multi-agent-mechanism]] — 三种多Agent机制总览
- [[claude-code-fork-subagent]] — Fork Subagent 缓存优化机制
- [[claude-code-coordinator-mode]] — Coordinator 并行协作模式
- [[dive-into-claude-code]] — 学术论文分析
- [[claude-code-permission-system]] — 权限系统（子代理权限覆盖相关）
- [[claude-code-session-persistence]] — 会话持久化（侧链转录相关）
- [[claude-code-context-compaction]] — 上下文管理（摘要返回是上下文策略）
- [[sub-agent-vs-team]] — 子代理 vs Agent 团队对比
- [[claude-code-swarm]] — Swarm 多 Agent 协作（Teammate + Mailbox）
- [[claude-code-memory-system]] — 记忆提取 Agent（forked agent 共享缓存）
- [[claude-code-builtin-agents]] — 5 个内置 Agent 完整 system prompt 与设计哲学
