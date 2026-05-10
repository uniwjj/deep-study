---
title: Claude Code 源码架构
description: Claude Code 架构全景——六抽象模型、黄金路径、10大设计模式、14+可迁移架构原则
aliases: [Claude Code架构, Claude Code源码解析, six abstractions]
tags: [ai-agent, tool, architecture]
sources: [2026/04/01/Claude Code 源码架构解析.md, 2026/05/10/Claude Code from Source.md, 2026/05/10/Claude Code 技术解析.html]
created: 2026-05-09
updated: 2026-05-10
---

# Claude Code 源码架构

Claude Code 是 Anthropic 的生产级 TypeScript 单体（~2000 文件），核心是一个 `while` 循环（调用模型 → 执行工具 → 重复），但 **98.4% 的代码**是围绕这个循环的操作基础设施。

## 六个核心抽象

| 抽象 | 文件 | 职责 |
|------|------|------|
| **查询循环** | `query.ts` (~1,730行) | async generator，全系统心跳。所有交互都经过它 |
| **工具系统** | `Tool.ts`, `tools.ts` | 40+ 工具，自我描述接口，14 步执行管道 |
| **任务系统** | `Task.ts`, `tasks/` | 后台工作单元，状态机 pending→running→completed/failed/killed |
| **状态** | `bootstrap/state.ts`, `state/store.ts` | 两层：进程单例（~80 字段）+ 响应式 store（34 行） |
| **记忆** | `memdir/` | 文件级记忆，LLM 侧查询召回 |
| **Hooks** | `hooks/` | 27 个生命周期事件，4 种执行类型 |

## 黄金路径：从按键到输出

```
用户输入 → REPL → query() generator
  → 模型流式输出
  → StreamingToolExecutor 在流式期间启动工具（推测执行）
  → 工具结果追加到消息历史
  → 再次调用模型（同一循环）
  → 模型不再产生工具调用时停止
```

三个关键洞察：
1. 查询循环是 generator——背压自然（消费者 pull），取消干净（`.return()`）
2. 工具执行与模型流式输出重叠——Read 调用在模型还在生成时就能完成
3. 整个循环是可重入的——没有单独的“处理工具结果”阶段

## 10 大设计模式

| # | 模式 | 说明 |
|---|------|------|
| 1 | **AsyncGenerator 作为 agent 循环** | 背压 + 取消 + 类型化终止状态 |
| 2 | **自我描述工具接口** | 每个工具声明自己的并发安全、权限、渲染 |
| 3 | **按访问模式分离状态** | 基础设施状态（单例） vs UI 状态（响应式） |
| 4 | **权限模式而非分散检查** | 7 种命名模式，统一解析链 |
| 5 | **递归 agent 架构** | 子 agent 是新实例化的相同循环 |
| 6 | **漏斗式启动** | 早退出快速路径，I/O 与初始化重叠 |
| 7 | **粘性锁存器缓存稳定** | `boolean | null` 三态锁定，防止 mid-session 缓存破坏 |
| 8 | **`DANGEROUS_` 命名规范** | 使破坏缓存的代码在 review 中可见 |
| 9 | **流式推测执行** | API 响应还在到达时就开始工具 |
| 10 | **按安全分区的并发批处理** | per-invocation 分类，保持顺序 |

## 系统架构图

```
用户 → 界面层 → Agent Loop → 权限系统 → 工具 → 执行环境
                    ↕
              状态与持久化
```

五层子系统：
```
表层（入口与渲染）→ 核心层（Agent Loop + 压缩管道）
→ 安全/操作层（权限+钩子+工具+沙箱+子代理）
→ 状态层（上下文装配+运行时+持久化+记忆+侧链）
→ 后端层（执行后端+外部资源）
```

## 可迁移的架构原则

1. **将复杂性推向边界** — 每个边界吸收混沌并输出秩序
2. **生成器循环优于回调** — 开发者拥有循环控制权
3. **文件存储优于数据库** — 透明、可审计、零基础设施
4. **工具自描述优于中央编排** — 添加工具 N+1 不需要修改现有代码
5. **Fork 代理实现缓存共享** — 成本优化本身就是能力
6. **Hook 优于插件** — 进程隔离，API contract 自 1971 年稳定

## 相关页面

- [[claude-code-bootstrap-pipeline]] — 5 阶段启动管道（300ms 预算）
- [[claude-code-state-architecture]] — 两层状态架构详细
- [[claude-code-api-layer]] — 多云提供商 + prompt cache
- [[claude-code-agent-loop]] — async generator 查询循环
- [[claude-code-tool-system]] — 14 步工具执行管道
- [[claude-code-concurrent-tools]] — 并发工具执行
- [[claude-code-memory-system]] — 文件级记忆系统
- [[claude-code-extensibility]] — Skills/Hooks 扩展机制
- [[claude-code-terminal-ui]] — 终端 UI 渲染架构
- [[claude-code-mcp]] — MCP 协议实现
- [[claude-code-performance]] — 性能优化体系
- [[dive-into-claude-code]] — MBZUAI 学术论文分析
- [[claude-code-permission-system]] — 7 种权限模式
- [[claude-code-context-compaction]] — 5 层压缩管道
- [[claude-code-subagent]] — 子代理委托
- [[claude-code-session-persistence]] — JSONL 会话持久化
- [[claude-code-sourcemap-reverse]] — Sourcemap 逆向还原
- [[claude-code-builtin-agents]] — 5 个内置 Agent 详解

## 10 大设计模式（实现级分析）

### 1. 双层 Generator 架构

**问题**：Agent 需同时处理会话状态管理（跨轮次记忆、计费）和行为循环（API/工具），职责耦合。

**方案**：外层 QueryEngine（AsyncGenerator）管理会话生命周期，内层 queryLoop 只关心一次请求的完整循环。`yield*` 传递消息。

**借鉴**：将 Agent 的"会话管理"和"行为执行"分离为两层——外层持久化/计费/SDK适配，内层纯状态机。

### 2. 流式工具并发执行

**问题**：等 API 响应结束再执行工具，延迟串行叠加。

**方案**：StreamingToolExecutor 在 API 流式过程中，tool_use 块一到达立即异步执行——读操作并发，写操作串行。

**借鉴**：每个工具声明 `isConcurrencySafe`，主循环维护执行槽位——空槽立即执行，满槽等待。读类工具几乎总是安全的。

### 3. 系统提示词分段注册表

**问题**：系统提示词是复杂动态文档，不同部分有不同的缓存需求。

**方案**：每个提示段由独立函数生成，通过 `systemPromptSection(name, fn)` 注册。静态段走全局 Prompt Cache，动态段每轮计算。BOUNDARY 标记精确分隔缓存域。

**借鉴**：明确区分"不变内容"（可全局缓存）和"会话相关内容"（动态生成）。使用注册表模式而非硬编码拼接。

### 4. 分层权限模型

**问题**：平衡自动化效率和用户安全——纯人工太慢，完全自动化太危险。

**方案**：12 步决策管道，部分检查"免疫绕过"（即使 bypassPermissions 也无法跳过）。5 种权限模式满足不同场景（开发 vs 生产）。

**借鉴**：区分三类规则：硬编码安全检查（永远运行）、用户可配置规则（Allow/Deny/Ask）、模式级别控制（开发/生产）。最危险操作无论如何都需人工确认。

### 5. 内建三条错误恢复路径

**问题**：生产环境中 API 错误多种多样（413/截断/过载），暴露给用户体验差。

**方案**：(1) 413 → Reactive Compact 自动摘要后重试；(2) output 截断 → 升级 64K + nudge 消息（最多 3 次）；(3) 模型过载 → 自动切换 fallback。

**借鉴**：为每种常见错误设计自动恢复策略。"扣留"（withhold）错误直到确认无法恢复。每种策略设置最大尝试次数，超限后优雅降级。

### 6. Token 预算与自动压缩

**问题**：长任务中对话历史无限增长，手动压缩打断工作流。

**方案**：每轮迭代前检查 token 数，超阈值调独立压缩 API 生成摘要替换原始历史。Token 预算不足时注入 nudge 消息促使 Agent 继续。

**借鉴**：在 Agent 主循环中定期检测上下文大小，提前触发压缩。设计可插拔的压缩策略（滑动窗口/LLM摘要/截断）。

### 7. 异步 Worker + task-notification 通信

**问题**：LLM 对话天生同步单线程，多 Agent 并行困难。

**方案**：主 Agent 通过 AgentTool 派发异步 Worker，Worker 结果以 XML 格式（task-notification）伪装成用户消息注入下一轮。SendMessage 向运行中 Worker 发追加指令。

**借鉴**：将 Worker 结果以结构化 XML/JSON 注入协调者消息流——最简单有效的多 Agent 通信方案。

### 8. MCP 工具动态注册与命名空间化

**问题**：第三方工具名可能冲突，连接状态随时变化。

**方案**：所有 MCP 工具前缀 `mcp__服务器名__工具名`。连接状态有明确状态机（connected/pending/failed/needs-auth/disabled）。子 Agent 可继承父的 MCP 配置。

**借鉴**：为第三方工具使用命名空间前缀。提供统一注册 API（工具名/描述/Schema/执行函数），让第三方工具与内置工具等价。

### 9. 系统提示词优先级覆盖链

**问题**：需同时支持多种定制场景（完全自定义 Agent、集成商定制、用户偏好、默认行为）。

**方案**：4 层覆盖链：overridePrompt → Coordinator 提示词 → Agent 定义 → 用户自定义 → 默认。每层可替换或追加。

**借鉴**：明确提示词优先级层次（框架默认 → 产品定制 → 用户偏好 → 实例覆盖）。"追加"比"替换"更安全（不丢失安全规则）。

### 10. Hooks 系统：工具调用前后的可编程干预

**问题**：不同部署场景（企业合规、安全审计、工作流集成）需在关键节点插入自定义逻辑。

**方案**：4 类 hooks（pre-tool-use / post-tool-use / user-prompt-submit / stop），以 shell 命令配置，通过 stdin/stdout 交互，语言无关。

**借鉴**：在 Agent 生命周期关键节点暴露 hooks。以进程/shell 命令实现而非代码注入。返回值应明确：allow/deny/modify。

## 核心源文件快速参考

| 机制 | 核心文件 | 关键函数/类 | 行数 |
|------|---------|------------|------|
| Agent 主循环 | `src/query.ts` | `queryLoop()` | 1729 |
| SDK 会话管理 | `src/QueryEngine.ts` | `QueryEngine.submitMessage()` | 1295 |
| 系统提示词 | `src/constants/prompts.ts` | `getSystemPrompt()` | 914 |
| 提示词优先级 | `src/utils/systemPrompt.ts` | `buildEffectiveSystemPrompt()` | 123 |
| 权限管道 | `src/utils/permissions/permissions.ts` | `hasPermissionsToUseToolInner()` | 1500+ |
| Bash 权限 | `src/tools/BashTool/bashPermissions.ts` | `bashToolHasPermission()` | 1400+ |
| 并发工具 | `src/services/tools/toolOrchestration.ts` | `runTools()`, `partitionToolCalls()` | 188 |
| 流式执行器 | `src/services/tools/StreamingToolExecutor.ts` | `addTool()`, `getCompletedResults()` | 530+ |
| Coordinator | `src/coordinator/coordinatorMode.ts` | `getCoordinatorSystemPrompt()` | 300+ |
| Agent 派生 | `src/tools/AgentTool/runAgent.ts` | `runAgent()` | 200+ |
| MCP 客户端 | `src/services/mcp/client.ts` | `connectToServer()`, `listTools()` | 3348 |
| Mailbox | `src/utils/teammateMailbox.ts` | `writeToMailbox()`, `readMailbox()` | 200+ |
