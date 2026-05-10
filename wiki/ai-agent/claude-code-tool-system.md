---
title: Claude Code 工具系统
description: Claude Code 的工具系统——40+ 工具实现、14步执行管道、7模式权限解析器、fail-closed默认值、per-invocation并发安全分类
aliases: [Tool System, 工具执行, buildTool, tool pipeline]
tags: [ai-agent, architecture, practice]
sources: [2026/05/10/Claude Code from Source.md, 2026/05/10/Claude-Code-Source-Analysis.pdf]
created: 2026-05-10
updated: 2026-05-10
---

# Claude Code 工具系统

工具系统涵盖 40+ 工具实现，中心化注册表，**14 步执行管道**，7 模式权限解析器，以及在模型完成响应之前就启动工具的流式执行器。

## 工具接口（~45 成员）

5 个关键成员：

| 成员 | 职责 |
|------|------|
| `call()` | 执行工具 |
| `inputSchema` | Zod schema，验证并生成 JSON Schema 用于 API |
| `isConcurrencySafe()` | 是否可以并行运行（接收解析后的输入） |
| `checkPermissions()` | 是否被允许 |
| `validateInput()` | 超越 schema 的语义验证 |

三个类型参数：`Tool<Input extends AnyObject, Output, P extends ToolProgressData>`

## buildTool() 的 Fail-Closed 默认值

```javascript
const SAFE_DEFAULTS = {
  isEnabled: () => true,
  isParallelSafe: () => false,   // 默认串行——fail-closed
  isReadOnly: () => false,        // 默认视为写操作
  checkPermissions: (input) => ({ behavior: 'allow', updatedInput: input }),
}
```

开发者忘记 `isConcurrencySafe` → 串行。忘记 `isReadOnly` → 视为写操作。忘记权限检查 → 默认允许（因为权限系统有独立层）。

## Per-Invocation 并发安全

`isConcurrencySafe(input: z.infer<Input>): boolean`

同一个工具，不同输入，不同安全分类：
- `ls -la` → 安全
- `rm -rf /tmp/build` → 不安全

**BashTool** 解析命令，将每个子命令对照已知安全的集合分类，仅当所有非中性部分都是 search/read 时返回 true。

## ToolResult 返回类型

```typescript
type ToolResult<T> = {
  data: T
  newMessages?: Message[]
  contextModifier?: (context: ToolUseContext) => ToolUseContext
}
```

`contextModifier` 仅在非并发安全工具上生效。对于并行工具，修饰器排队直到批次完成后应用。

## 14 步执行管道

`checkPermissionsAndCallTool()`：

**验证（步骤 1-4）**：
1. 工具查找（回退到 `getAllBaseTools()` 处理旧会话别名）
2. Abort 检查（防止 Ctrl+C 前排队的调用浪费计算）
3. Zod 验证（类型不匹配时追加 ToolSearch 提示）
4. 语义验证（FileEditTool 拒绝 no-op 编辑；BashTool 阻止 MonitorTool 可用时的 standalone `sleep`）

**准备（步骤 5-6）**：
5. 推测性分类器启动（Bash 命令 kick off auto-mode 安全分类器）
6. 输入回填（扩展路径如 `~/foo.txt` → 绝对路径以保持转录稳定性）

**权限（步骤 7-9）**：
7. PreToolUse hooks（可决定、修改输入、注入上下文或停止）
8. 权限解析（hook 决定优先；否则走规则匹配 → 工具特定检查 → 模式默认 → 交互提示）
9. 权限拒绝处理（构建错误消息，执行 PermissionDenied hooks）

**执行与清理（步骤 10-14）**：
10. 工具执行（`call()` with 原始输入）
11. 结果预算（超大输出持久化到 `~/.claude/tool-results/{hash}.txt`）
12. PostToolUse hooks（修改 MCP 输出或阻止继续）
13. 新消息追加（子 agent 转录、系统提醒）
14. 错误处理（为遥测分类错误）

## 工具结果预算

| 工具 | maxResultSizeChars | 原因 |
|------|-------------------|------|
| BashTool | 30,000 | 大多数有用输出足够 |
| FileEditTool | 100,000 | diff 可以很大 |
| GrepTool | 100,000 | 带上下文的搜索结果 |
| FileReadTool | Infinity | 通过自己的 token 限制自我边界 |

超阈值时：完整内容保存到磁盘，替换为预览。模型可以 Read 访问完整输出。

## 权限系统：7 种模式

| 模式 | 行为 |
|------|------|
| `default` | 工具特定检查；无法识别时提示用户 |
| `acceptEdits` | 自动允许文件编辑；其他操作提示 |
| `plan` | 只读——拒绝所有写操作 |
| `dontAsk` | 自动拒绝通常需要提示的操作 |
| `bypassPermissions` | 允许所有操作无需提示 |
| `auto` | 转录分类器决定（feature-flagged） |
| `bubble` | 内部模式——升级到父 agent |

## 个别工具详情

**BashTool**（最复杂）：
- `splitCommandWithOperators()` 分解复合命令
- 对照 BASH_SEARCH_COMMANDS、BASH_READ_COMMANDS 分类
- `_simulatedSedEdit`：在沙箱中预计算 sed 结果，注入到输入——保证预览的结果

**FileEditTool**：
- 集成 `readFileState` LRU 缓存
- 过期检测——如果文件在模型上次读取后被修改则拒绝编辑

**FileReadTool**：
- `maxResultSizeChars: Infinity`——如果 Read 输出持久化到磁盘，模型需要 Read 持久化文件（无限循环）
- 阻止危险的设备路径（`/dev/zero`、`/dev/random`）

**ToolUseContext**：~40 个字段的上下文对象，按关注点分组（配置、执行状态、UI 回拨、agent 上下文）。

## 注册表架构

`assembleToolPool()`：内置工具 + MCP 工具合并

关键设计：排序后拼接而非混合排序，保留 prompt cache 断点在最后一个内置工具之后。

## 设计模式

- **Fail-closed 默认值**——忘记标志的开发人员获得安全行为
- **输入依赖安全分类**——同一工具，不同输入，不同安全配置
- **分层权限**——没有单层机制足够；6 层解析链
- **结果预算**——每个工具的限制防止单次爆炸
- **遥测安全的错误分类**——提取安全字符串，不记录原始错误消息

## 相关页面

- [[claude-code-architecture]] — 六抽象整体架构
- [[claude-code-agent-loop]] — 查询循环（工具调用者）
- [[claude-code-concurrent-tools]] — 并发工具执行
- [[claude-code-permission-system]] — 权限系统详解
- [[claude-code-mcp]] — MCP 工具集成
- [[claude-code-search-strategy]] — 搜索工具组合策略（Grep/Glob/Read）

## 4 能力原语

所有工具归于 4 个能力维度：

| 原语 | 动作 | 代表工具 |
|------|------|---------|
| **Read** | 读取信息 | FileRead, Grep, Glob, WebFetch, WebSearch |
| **Write** | 创建或修改 | FileWrite, FileEdit, NotebookEdit |
| **Execute** | 运行命令 | Bash, PowerShell, REPL |
| **Connect** | 连接外部系统 | MCP tools, Agent, SendMessage, TeamCreate |

权限分级：Read = 低风险，Write/Execute = 中高风险，Connect = 外部交互风险。

## Deferred Tools & ToolSearch

系统初始仅告知模型工具**名称**（不含完整参数定义）。模型需要时先调 `ToolSearch` 获取完整定义。MCP 工具定义占 ~4000-6000 token——延迟加载省大量上下文。

匹配算法：
- **Exact select**：`select:Read,Edit,Grep` 精确选择
- **Keyword search**：驼峰分词（`FileEditTool` → `['file','edit','tool']`），MCP 名解析（`mcp__slack__send_message` → `['slack','send','message']`）
- 支持 `+` 前缀强制关键词
- 最终分数结合：工具名匹配 + 描述关键词匹配 + `searchHint`

## Bash 安全分析引擎

两层安全分析：
1. `commandSemantics.ts`：语义分类——分类管道中的每个命令（search: find/grep/rg；read: cat/head/tail/jq；list: ls/tree；neutral: echo/printf）。仅当所有部分都是 search/read 时管道才被分类为只读
2. `bashSecurity.ts`：AST 级解析——检测输出重定向、命令替换和隐藏的危险模式
