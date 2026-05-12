---
title: OpenCode 源码架构
description: OpenCode 全架构分析——开源 AI 编程智能体，MIT 协议，provider-agnostic，20+ LLM 提供商，monorepo 多端架构（TUI/Desktop/IDE/Web）
aliases: [opencode, OpenCode]
tags: [ai-agent, tool, architecture]
sources: [2026/05/12/deepwiki-opencode.html]
created: 2026-05-12
updated: 2026-05-12
---

# OpenCode 源码架构

OpenCode 是 100% 开源（MIT）的 AI 编程智能体，作为 Claude Code、GitHub Copilot 等商业方案的替代。核心定位：**provider-agnostic（模型无关）**、多端可用、高度可扩展。

## 核心特征

| 特征 | 说明 |
|------|------|
| **开源协议** | MIT |
| **模型提供商** | 20+：OpenAI、Anthropic、Google、Amazon Bedrock、Azure、Groq、Together AI、Cerebras、Mistral、xAI 等 |
| **本地模型** | Ollama、llama.cpp、LM Studio |
| **多端接入** | TUI 终端 / Desktop（Electron/Tauri）/ VS Code & Zed 插件 / Web 文档 |
| **运行时** | Bun 包管理 + Effect 函数式运行时 |
| **多 Agent** | Task Tool 子代理委托 |
| **LSP** | 内置 20+ 语言服务器，自动下载 |
| **MCP** | 完整支持，含 OAuth 认证 |
| **分发** | npm、Homebrew、Pacman、AUR、Chocolatey、Scoop、Nix、Docker |

## 仓库结构（Monorepo）

```
根目录（Bun workspaces）
├── packages/
│   ├── opencode/          # 核心：CLI、HTTP 服务、TUI、工具、Provider（主入口）
│   ├── sdk/js/            # TypeScript SDK（@opencode-ai/sdk）
│   ├── plugin/            # 插件开发包（@opencode-ai/plugin）
│   ├── app/               # 共享 UI 业务逻辑（SolidJS）
│   ├── ui/                # 组件库（消息渲染、diff 查看器）
│   ├── desktop/           # 桌面端（Electron/Tauri）
│   ├── web/               # Astro 文档站
│   ├── console/{core,app,function,mail}/  # SaaS 控制台（Drizzle + Stripe）
│   ├── enterprise/        # 企业版
│   ├── function/          # 函数服务
│   └── slack/             # Slack 集成
├── sdks/
│   └── vscode/            # VS Code 扩展
└── infra/                 # 基础设施代码
```

核心包依赖关系：`opencode` ← `sdk/js` ← `plugin` ← 所有客户端

## 核心架构

### Client-Server 分层

```
IDE/VS Code  →→  ACP Protocol (stdio JSON-RPC)  →→┐
TUI Worker   →→  内部 SDK（同进程）               →→├→  Hono HTTP Server (REST + SSE)
Desktop App  →→  HTTP SDK（远程）                 →→┘      ↓
                                                      Session & Agent Engine
                                                      ↓
                                                   Tool System + Provider + LSP + MCP
```

- **Hono Server**：基于 Effect 框架的 HTTP 服务器，暴露 REST + SSE 端点
- **SDK Transport**：统一抽象，支持同进程（进程内调用）和远程（HTTP）两种模式
- **v2 API**：OpenAPI 代码生成的下一代客户端

### 技术栈

| 层级 | 技术 |
|------|------|
| HTTP 框架 | Hono + Effect HttpApi |
| AI SDK | Vercel AI SDK（`ai` + `@ai-sdk/*`） |
| 函数式运行时 | Effect（`effect` + `@effect/platform-node`） |
| UI | SolidJS + OpenTUI |
| 数据库 | Drizzle ORM + SQLite |
| 包管理 | Bun（workspaces + catalog） |

## 子系统详解

### 1. CLI 入口（`packages/opencode/src/index.ts`）

- 使用 `yargs` 解析命令行参数
- **全局选项**：`--print-logs`、`--log-level`、`--pure`（无外部插件）
- **中间件管道**：日志初始化 → 数据库迁移 → 命令路由
- 支持 `--` 参数分隔符

### 2. 配置系统（8 层优先级）

从低到高依次加载合并（深合并，数组字段特殊处理）：

| 优先级 | 来源 | 说明 |
|--------|------|------|
| 1 | Remote Config | `.well-known/opencode` URL |
| 2 | Global Config | 用户级 `~/.config/opencode/` |
| 3 | Custom Config | `OPENCODE_CONFIG` 环境变量指定路径 |
| 4 | Project Config | `opencode.json`（项目根目录） |
| 5 | `.opencode` 目录 | 项目本地 config、agents、plugins |
| 6 | Inline Config | `OPENCODE_CONFIG_CONTENT` 环境变量 |
| 7 | Account Config | 控制台组织设置 |
| 8 | Managed Config | 企业管理员控制（最高优先） |

### 3. 会话与 Agent 系统

**Session**（会话）：
- 使用降序 ULID 作为 ID（天然逆序时间排序）
- 支持父子会话（fork）
- SQLite + Drizzle ORM 持久化
- 含文件增删改统计摘要

**Agent**：
- 内置 Agent 类型：`build`、`plan`、`explore` 等
- 每个 Agent 有独立配置（模型、工具权限、指令）
- Agent 循环：用户输入 → LLM 推理 → 工具执行 → 结果反馈 → 循环

**Prompt 模板**：按模型 ID 分发 6 套模板（anthropic.txt、beast.txt 等）

### 4. AI Provider 系统

**统一的 Provider 抽象层**（基于 Vercel AI SDK）：

| Provider | 包 | 说明 |
|----------|-----|------|
| Anthropic | `@ai-sdk/anthropic` | Claude 系列 |
| OpenAI | `@ai-sdk/openai` | GPT 系列 |
| Google | `@ai-sdk/google` | Gemini |
| Amazon Bedrock | `@ai-sdk/amazon-bedrock` | AWS 托管模型 |
| OpenRouter | `@openrouter/ai-sdk-provider` | 统一网关 |
| Azure | `@ai-sdk/azure` | Azure OpenAI |
| xAI | `@ai-sdk/xai` | Grok |
| GitHub Copilot | `@ai-sdk/github-copilot` | Copilot 模型 |

**认证方式**：API Key（存 `~/.local/share/opencode/auth.json`）+ OAuth 流程

### 5. 工具系统与权限

**工具定义**：`Tool.define()` API，Effect Generator 模式，自动参数校验（Zod Schema）

**Tool.Context** 关键成员：

| 成员 | 类型 | 用途 |
|------|------|------|
| `sessionID` | SessionID | 当前会话 |
| `messageID` | MessageID | 父消息 ID |
| `agent` | string | 活跃 Agent 名 |
| `abort` | AbortSignal | 取消信号 |
| `messages` | MessageV2[] | 完整对话历史 |
| `ask(req)` | Function | 权限门控（allow/deny/ask） |

**内置工具**：`bash`、`edit`、`read`、`write`、`grep`、`glob`、`task`

### 6. HTTP 服务与事件总线

- **Hono + Effect HttpApi**：类型安全的 REST API，自动生成 OpenAPI 文档
- **SSE**（Server-Sent Events）：实时事件推送到 UI 客户端
- **Pub/Sub 事件系统**：`Bus`（实例级）+ `GlobalBus`（全局单例），基于 Effect PubSub
- **SyncEvent**：持久化事件 + 序列追踪 + projector 模式（先更新 DB，再广播）

### 7. LSP 与格式化（20+ 语言）

- 每个 LSP Server 定义：`id` + `extensions` + `root detection` + `spawn`
- **NearestRoot** 策略：从文件向上搜索标记文件
- **自动下载**：缺失的 LSP 服务器自动安装到 `Global.Path.bin`
- 代码格式化：编辑后自动应用语言特定格式化

### 8. 插件系统

三种加载模式：

| 模式 | 来源 | 配置位置 |
|------|------|----------|
| Internal | 静态导入 | `INTERNAL_PLUGINS` 常量 |
| npm | Bun 下载安装 | `opencode.json` plugin 数组 |
| Local | 文件系统扫描 | `.opencode/plugins/` 或 `~/.config/opencode/plugins/` |

插件接口：`Plugin(ctx: PluginInput) => Hooks`
Hooks 钩子：工具执行前后、LLM 参数解析时、系统事件回调

### 9. MCP 集成

完整实现 Model Context Protocol：

- **Server 类型**：本地子进程 + 远程 HTTP/SSE
- **连接状态**：`connected | disabled | failed | needs_auth | needs_client_registration`
- **能力暴露**：MCP 工具 → AI Tool（`convertMcpTool`）、MCP Prompts → 会话命令、MCP Resources → 资源引用
- **OAuth 认证**：完整 OAuth 流程支持
- **配置**：`opencode.json` 的 `mcp` 字段

### 10. Skills & Command 系统

| 概念 | 说明 |
|------|------|
| **Commands** | Prompt 模板，支持变量替换（`$ARGUMENTS`、`$1`），源自内置/配置/MCP/Skills |
| **Skills** | 领域特定指令集（SKILL.md），由 Skill.Service 动态加载 |
| **Specialized Agents** | 有特定 prompt 和工具权限的 AI 配置 |

调用方式：`/command-name`（TUI 中）

### 11. ACP 协议（Agent Client Protocol）

- 基于 **JSON-RPC** 的编辑器集成标准
- 通过 **stdio** 通信
- 支持 Zed、JetBrains、Neovim 等编辑器
- 实现：`packages/opencode/src/acp/`，基于 `@agentclientprotocol/sdk`
- 入口命令：`opencode acp`

### 12. LLM Core 库（`packages/llm`）

独立的 Effect 函数式 LLM 客户端库，提供统一的 schema-first 接口：

| 协议文件 | 适配器 | 特性 |
|----------|--------|------|
| `anthropic-messages.ts` | anthropic-messages | 交错思考、临时缓存提示、工具流式 |
| `openai-responses.ts` | openai-responses | HTTP + WebSocket 双传输 |
| `bedrock-converse.ts` | bedrock-converse | SigV4 签名（aws4fetch） |
| `gemini.ts` | gemini | Google 严格 JSON Schema 清理 |

核心流程：`LLMRequest` → 协议处理器（lowering） → Provider API → 原始响应流 → lifting → `LLMEvent` 流

## 与 Claude Code 的关键差异

参见 [[claude-code-vs-opencode]] 的四维度详细对比。核心差异总结：

| 维度 | Claude Code | OpenCode |
|------|-------------|----------|
| 上下文管理 | 三层递进压缩 | 单层 overflow（22行） |
| Prompt | 静/动态分割 + cache | 6 套模板，无 cache |
| 多 Agent | SubAgent/Fork/Teammate | 仅 Task Tool |
| 工具编排 | 自动并行分区 + 安全验证 | Batch Tool 模式 |

OpenCode 优势：75+ 模型提供商、完整 TUI、开源 120K+ star、MIT 协议。

## 相关页面

- [[claude-code-vs-opencode]] — 四维度深度对比（上下文/Prompt/Agent/工具）
- [[claude-code-architecture]] — Claude Code 架构参照
- [[agent-mcp-protocol]] — MCP 协议原理
- [[agent-skills-system]] — Skills 系统
- [[agent-architecture-patterns]] — Agent 架构模式
