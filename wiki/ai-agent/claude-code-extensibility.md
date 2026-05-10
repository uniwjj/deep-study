---
title: Claude Code 可扩展性
description: Claude Code 的四种扩展机制——MCP服务器、插件、Skills和Hooks——按上下文成本分层，Skills两阶段加载，Hook快照安全模型，27个生命周期事件
aliases: [可扩展性, extensibility, MCP, skills, hooks, plugins]
tags: [ai-agent, concept, architecture]
sources: [2026/05/10/Dive into Claude Code.txt, 2026/05/10/Claude Code from Source.md]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 可扩展性

核心分离：**Skills 扩展模型能做什么**（内容，以指令注入）。**Hooks 扩展何时和如何执行**（控制流，生命周期拦截器）。

## 为什么需要四种机制？

| 机制 | 唯一能力 | 上下文成本 | 注入点 |
|------|---------|-----------|--------|
| **MCP 服务器** | 外部服务集成（多传输） | 高（工具模式） | `model()`: 工具池 |
| **插件** | 多组件打包+分发 | 中（变化） | 全部三个注入点 |
| **Skills** | 领域特定指令+元工具调用 | 低（仅描述） | `assemble()`: 上下文注入 |
| **Hooks** | 生命周期拦截+事件驱动自动化 | 默认为零 | `execute()`: 工具调用前后 |

## Skills：两阶段加载

### Phase 1（启动时）
读取每个 `SKILL.md`，提取 frontmatter 元数据到 system prompt。

### Phase 2（调用时）
加载完整 skill 内容并进行变量替换。

### 七个来源优先级

| 优先级 | 来源 | 位置 |
|--------|------|------|
| 1 | Managed (Policy) | 企业控制 |
| 2 | User | `~/.claude/skills/` |
| 3 | Project | `.claude/skills/` |
| 4 | Additional Dirs | `--add-dir` flag |
| 5 | Legacy Commands | `.claude/commands/` |
| 6 | Bundled | 编译进二进制 |
| 7 | MCP | MCP 服务器提示 |

### MCP 安全边界
MCP skills **永不执行内联 shell 命令**。信任边界是绝对的。

### SkillTool vs AgentTool
- **SkillTool**：将指令注入**当前上下文窗口**
- **AgentTool**：生成**新的隔离上下文窗口**

## Hooks：四种用户可配置类型

| 类型 | 机制 |
|------|------|
| Command hooks | 生成 shell 进程 |
| Prompt hooks | 单次 LLM 调用 |
| Agent hooks | 多回合 agentic 循环（**最大 50 回合**） |
| HTTP hooks | POST 到 URL |

### 退出码语义

| 退出码 | 含义 | 是否阻塞 |
|--------|------|---------|
| 0 | 成功 | 否 |
| 2 | 阻塞错误（stderr 显示） | 是 |
| 其他 | 非阻塞警告 | 否 |

注意：exit 2 是语义化阻塞码，**不是** exit 1。

### 五个最关键生命周期事件

| 事件 | 时机 | 能力 |
|------|------|------|
| **PreToolUse** | 每次工具执行前 | 阻止、修改输入、自动批准 |
| **PostToolUse** | 成功执行后 | 注入上下文、替换输出 |
| **Stop** | Claude 得出结论前 | 阻塞 hook 强制继续 |
| **SessionStart** | 会话开始 | 设置环境变量 |
| **UserPromptSubmit** | 用户提交消息 | 阻止处理 |

### 快照安全模型（TOCTOU 防护）

- `captureHooksConfigSnapshot()` 在**启动时调用一次**
- `executeHooks()` 只从**快照**读取，永不复读设置文件
- 防止 hook 配置的 TOCTOU（Time-of-Check-Time-of-Use）攻击

### Skills-Hooks 集成

当 skill 被调用，其 frontmatter 声明的 hooks 注册为**会话作用域 hooks**。`skillRoot` 变为 `CLAUDE_PLUGIN_ROOT` 供 hook 的 shell 命令使用。

## 相关页面

- [[claude-code-architecture]] — 六抽象整体架构
- [[claude-code-mcp]] — MCP 协议实现
- [[claude-code-tool-system]] — PreToolUse/PostToolUse 在工具管道中的位置
- [[claude-code-memory-system]] — 记忆系统（另一种扩展形式）

## 工具池组装

`assembleToolPool()` 五步管道（`tools.ts`）：

1. **基础工具枚举** — 最多 54 个工具（19 个无条件 + 35 个条件）
2. **模式过滤** — CLAUDE_CODE_SIMPLE 模式仅 3 个工具（Bash, Read, Edit）
3. **拒绝规则预过滤** — 被拒绝的工具从模型视角完全移除
4. **MCP 工具集成** — MCP 工具经拒绝规则过滤后与内置工具合并
5. **去重** — 按名称去重，内置工具优先于 MCP 工具

## Agent Loop 中三种注入点

```
assemble() → 控制模型看到什么 → CLAUDE.md, Skills, MCP资源
model()    → 控制模型能调用什么 → 内置工具, MCP工具, SkillTool, AgentTool
execute()  → 控制操作是否/如何运行 → 权限规则, Hooks
```

## 相关页面

- [[dive-into-claude-code]] — 论文总览
- [[claude-code-permission-system]] — 权限系统（Hooks 与权限管道深度集成）
- [[claude-code-context-compaction]] — 上下文管理
- [[claude-code-subagent]] — 子代理（AgentTool 元工具）
