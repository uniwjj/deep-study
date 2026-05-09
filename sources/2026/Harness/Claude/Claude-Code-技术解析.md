---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-harness]
---

# Claude Code 核心实现机制 — 技术解析合集

**文章信息**：
- **原文来源**：https://skyexu.github.io/claude-source-learning/
- **分析版本**：`@anthropic-ai/claude-code v2.1.88`
- **源文件规模**：1884 个 TS/TSX 源文件，40+ 内置工具

---

**一段话总结**：

本系列从源码层面深度解析 Claude Code 的五大核心子系统：以 `while(true)` 无限循环驱动的双层 Agent 执行引擎（QueryEngine + queryLoop）、由 14 个函数段动态拼接的分段式系统提示词（通过 BOUNDARY 标记分离全局缓存与会话缓存）、12 步顺序决策的工具权限管道（含 5 种权限模式与免疫绕过机制）、基于 Coordinator 模式的多 Agent 协作架构（AgentTool 派发、task-notification 回传、Mailbox 文件通信），以及 5 个内置专用 Agent（Explore/Plan/Verification/General Purpose/Guide）。文章最终提炼出 10 个可直接应用到自建 Agent 系统的工程设计模式。

---

**作者核心判断**：

Claude Code 的核心工程哲学体现在三个维度：**安全性通过免疫绕过的分层权限保障**（部分安全检查在任何模式下都无法跳过）、**效率通过流式并发工具执行和 Prompt Cache 实现**（API 流式传输过程中即开始执行工具）、**可扩展性通过注册表模式和优先级覆盖链实现**（自定义 Agent 与内置 Agent 完全等价）。这套架构设计在复杂的 Agent 场景下实现了"大多数错误用户无感知"的自愈能力，是构建生产级 Agent 系统的重要参考。

---

## 一、整体架构

Claude Code 是一个多层次 Agent 系统，核心是无限循环的 query 引擎，围绕其构建了工具系统、权限体系和多 Agent 协调层。

**五大核心子系统**：

- **Agent 主循环**：`query.ts`（1729行）中的 `while(true)` 主循环，驱动 API 流式调用、工具执行、上下文更新
- **系统提示词**：`constants/prompts.ts`（914行），14 段函数动态拼接，静态段走全局 Prompt Cache
- **工具权限控制**：12 步决策管道，5 种权限模式，沙箱集成，AI 分类器辅助
- **多 Agent 协作**：Coordinator 模式 + AgentTool 异步派生 + Mailbox 文件系统通信
- **内置 Agent**：5 个专用 Agent，各有独立模型、工具白名单和系统提示词

**MCP 集成**：MCP 服务器通过 ListTools RPC 动态暴露工具，名称规范化为 `mcp__server__tool`，子 Agent 可继承父 Agent 的 MCP 客户端或拥有独立配置。

---

## 二、Agent 主循环

### 双层架构

**外层 QueryEngine**（`QueryEngine.ts` · 1295行）负责会话生命周期管理：
- 跨轮次消息历史 `mutableMessages`
- SDK 输出流（yield SDKMessage）
- Token 使用量与总费用跟踪
- 消息转录持久化（sessionStorage）
- USD 预算限制检查
- 权限拒绝记录（permissionDenials）

**内层 queryLoop**（`query.ts` · 1729行）驱动实际行为循环：
- `while(true)` 无限循环主体
- 流式调用 Anthropic API
- 提取并执行 tool_use 块
- 自动压缩 / Context Collapse
- 错误恢复（prompt_too_long / max_tokens）
- Token 预算 nudge 继续机制

**工具执行层 StreamingToolExecutor**：
- 工具在 API 流式完成前已开始执行
- 并发安全工具批量并行（最多 10 个）
- 非并发工具（写文件、Bash）严格串行
- `isConcurrencySafe()` 每个工具自声明

### 主循环完整流程

每次迭代经历 5 个关键节点，实现完整的"思考-行动-观察"周期：

0. **迭代前准备**：AutoCompact 检查、snip 压缩、Context Collapse、Token 阻塞判断
1. **流式调用 Anthropic API**：`for await (message of deps.callModel(...))` — tool_use 块到达即投入 StreamingToolExecutor
2. **工具执行**：`runTools(toolUseBlocks, ...)` — 按并发安全性分批执行（读类并行 / 写类串行）
3. **终止条件检查**：AbortController 中断 · maxTurns 限制 · Stop Hooks 阻止 · Token 预算耗尽
4. **QueryEngine 消费输出**：转录持久化、构建 SDKMessage、yield result

### 循环终止条件

| 终止原因 | 触发条件 |
|---------|---------|
| `completed` | 助手最终消息无 tool_use 块 |
| `aborted_streaming` | AbortController 信号触发 |
| `max_turns` | 达到 maxTurns 上限 |
| `blocking_limit` | Token 超过阻塞阈值且无法压缩 |
| `stop_hook_prevented` | 外部 stop hook 返回阻止信号 |

### 三条错误恢复路径

**Reactive Compact（响应式压缩）**：API 返回 413 prompt_too_long → 临时扣留错误消息 → 调用独立压缩 API 摘要历史 → 重试原请求。仅尝试一次，失败则返回原始错误。

**Max Output Tokens 恢复**（最多 3 次重试）：第一次升级到 64k max_tokens（ESCALATED_MAX_TOKENS），之后注入 meta 消息 `"Output token limit hit. Resume directly—no apology..."` 驱动模型继续。

**模型 Fallback 切换**：API 抛出 FallbackTriggeredError → 自动切换 fallbackModel → 清空当前轮次消息（Tombstone）→ 注入系统通知告知用户。

### Task 状态机（5 态）

`pending` → `running` → `completed`（终止态✓） / `failed`（终止态✗） / `killed`（终止态⊗）

一旦进入终止态，`isTerminalTaskStatus()` 判断后不再转换，从 AppState 清理。

---

## 三、系统提示词

### 分段构建策略

`getSystemPrompt()` 按优先级组装 14 个函数段，通过 `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` 标记精确分割两个缓存域：

**静态段（全局 Prompt Cache，约 70% 总量）**：跨所有用户和会话共享，走 `cacheScope: 'global'`
**动态段（会话/用户级缓存）**：通过 `systemPromptSection()` 注册表管理，按需计算

### 14 个提示词段详解

**① 角色定位（静态）** — `getSimpleIntroSection()`：定义 Claude Code 身份，注入网络安全声明和 URL 生成限制（禁止猜测 URL，防止幻觉链接）

**② 系统规则（静态）** — `getSimpleSystemSection()`：GFM markdown 输出规范、权限模型说明、提示注入警告、自动上下文压缩通知；外部版额外说明 Hooks 机制（Git hooks 输出视为用户反馈）

**③ 任务执行规范（静态）** — `getSimpleDoingTasksSection()`：
- 不额外添加功能、不过度抽象、不添加不必要错误处理
- 默认不写注释（除非 WHY 不明显）
- 发现用户误解时直接指出（协作者而非执行者）
- 诚实报告测试结果（不声称"全部通过"）

**④ 操作谨慎规则（静态）** — `getActionsSection()`：区分可逆操作（编辑文件/运行测试）与高风险操作（删除/强制推送/发布包/发送消息），引入"blast radius（爆炸半径）"评估概念

**⑤ 工具使用规范（静态）** — `getUsingYourToolsSection(enabledTools)`：
- 专用工具优先（Read/Edit/Write/Glob/Grep 替代 Bash）
- 最大化并行工具调用
- 用 TodoWrite 跟踪任务进度（立即标记完成）

**⑥ 语气与风格（静态）** — `getSimpleToneAndStyleSection()`：禁用 emoji（除非用户明确要求）、简洁回应、代码引用格式（`file_path:line_number`）

**⑦ 输出效率（静态）** — `getOutputEfficiencySection()`：直接说答案，跳过废话和铺垫；内部版（ANT）额外要求倒金字塔结构、匹配读者专业水平

══ `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` ══（此处分割全局缓存域）

**⑧ 会话特定指引（动态）** — `getSessionSpecificGuidanceSection()`：根据启用工具集动态生成（何时用子 Agent / Fork 模式 / 验证 Agent / 技能快捷方式）

**⑨ 项目记忆（动态）** — `loadMemoryPrompt()`：扫描当前目录及父目录（最多 5 层）的 CLAUDE.md 文件，直接注入内容；文档即配置，Markdown 格式支持团队协作

**⑩ 运行时环境信息（动态）** — `computeSimpleEnvInfo(model)`：注入工作目录、Git 状态、平台、Shell、OS 版本、模型名、知识截止日期、可用模型 ID 列表

**⑪ 语言偏好（动态）** — `getLanguageSection()`：根据 settings.json 的 language 配置注入强制回复语言，技术术语保持原文形式

**⑫ MCP 服务器自定义指令（动态禁止缓存）** — `getMcpInstructionsSection()`：注入各 MCP 服务器的 instructions 元数据；标记为 `DANGEROUS_uncachedSystemPromptSection`，防止服务器断连后缓存过期

**⑬ 暂存目录指令（动态）** — `getScratchpadInstructions()`：为每个会话创建独立临时目录（如 `/tmp/claude-scratchpad-{sessionId}`），实现会话隔离

**⑭ Token 预算机制（动态）** — `TOKEN_BUDGET` feature：用户设定目标 token 数，每轮显示已用量，未达目标时自动注入 nudge 消息续写，支持超大任务输出

### 四层优先级覆盖系统

`buildEffectiveSystemPrompt()` 实现 4 层优先级（高→低）：

| 优先级 | 类型 | 触发场景 |
|--------|------|---------|
| 0（最高）| overrideSystemPrompt | Loop 模式 / SDK 完全定制 |
| 1 | Coordinator 提示词 | `CLAUDE_CODE_COORDINATOR_MODE=1` |
| 2 | agentSystemPrompt | mainThreadAgentDefinition 设置时 |
| 3 | customSystemPrompt | `--system-prompt` CLI 参数 |
| 4（默认）| defaultSystemPrompt | 完整 14 段默认提示词 |

`appendSystemPrompt` 无论哪层都追加到末尾（任何情况下都生效）。

### 额外上下文注入机制

**userContext**：以 `<user-context>` XML 标签包裹，插入到每次请求用户消息前，包含 Git 状态、当前分支、近期提交等项目上下文

**systemContext**：以 `<system-context>` XML 标签追加到 system prompt 末尾，包含实时 token 使用量、当前轮次、是否接近上下文限制、预算百分比

**memoryMechanicsPrompt**：SDK 嵌入场景下，通过 `Memory.write()` / `Memory.read()` 工具操作 MEMORY.md 实现跨会话持久化记忆

---

## 四、工具权限控制

### 5 种权限模式（Shift+Tab 循环切换）

| 模式 | 行为 |
|------|------|
| `default` | 未明确允许的操作向用户弹出确认框 |
| `acceptEdits` | 自动接受工作目录内文件编辑，其余仍需确认 |
| `plan` | 先审查执行计划，再批准实际操作 |
| `bypassPermissions` | 自动允许所有操作（需 Statsig 门控） |
| `auto` | AI 分类器（TRANSCRIPT_CLASSIFIER）决策 |

### 12 步权限决策管道

每次工具调用经过 `hasPermissionsToUseToolInner()` 完整决策链（来自 `permissions.ts` 1158-1319行）：

- **1a** — 整个工具在 Deny Rule？→ **DENY**
- **1b** — 整个工具有 Ask Rule？→ **ASK**
- **1c/1d** — 工具自身 `checkPermissions()` 返回 DENY？→ **DENY**
- **1e** — 工具声明 `requiresUserInteraction()`？→ **ASK（免疫绕过）**
- **1f** — 内容级 Ask Rule 匹配（如 `Bash(npm publish:*)`）？→ **ASK（免疫绕过）**
- **1g** — 安全路径保护（`.git/`、`.claude/`、`.bashrc/.zshrc` 写操作）？→ **ASK（免疫绕过）**
- **2a** — bypassPermissions 模式？→ **ALLOW**
- **2b** — 整个工具在 Allow 白名单？→ **ALLOW**
- **2c** — 内容级 Allow Rule 匹配？→ **ALLOW**
- **3** — passthrough → 进入 `useCanUseTool.tsx` 交互流程 → **ASK**

### 规则系统

**alwaysAllowRules**（自动放行）：`Bash(git status:*)`、`Read`、`Edit(/src/**:*)`

**alwaysDenyRules**（硬拒绝）：`Bash(rm -rf:*)`、`Bash(npm publish:*)`、`WebSearch`

**alwaysAskRules**（强制确认，免疫绕过）：`Bash(git push:*)`、`Bash(curl * | bash:*)`

**Bash 规则匹配逻辑**（`bashPermissions.ts`）：`stripSafeWrappers()` 去除 timeout/nice/nohup → `splitCommandWithOperators()` 拆分复合命令（&&、|、;）→ 逐子命令匹配规则

### AI 分类器 + 拒绝跟踪

在 `auto` 模式下，分类器连续拒绝超 **3 次** 或累计拒绝超 **20 次** → 自动降级为交互式弹窗，避免 Agent 被无限拒绝卡死。

### Headless Agent 权限路径

`shouldAvoidPermissionPrompts = true`（异步子 Agent、SDK 非交互会话）时：先尝试 PermissionRequest hooks → Hooks 允许则放行 → 无响应则自动拒绝 → 分类器超限则 AbortError。

---

## 五、多 Agent 架构

### 三种 Agent 执行模型

**Local Async Agent**（后台派生）：通过 `AgentTool()` 派生，TaskID 格式 `a-{16HEX}`，结果通过 `<task-notification>` XML 回传，可通过 `SendMessageTool` 继续对话，继承父 Agent 工具集（过滤交互式工具），支持 `isolation: 'worktree'` 沙箱执行

**In-Process Teammate**（Team Swarm）：身份标识 `agentName@teamName`，通过 AsyncLocalStorage 上下文隔离，通过 Mailbox 文件系统通信，Leader 可广播消息给所有 Teammate

**Remote Agent**：部署到 CCR（Claude Code Remote），TaskID 格式 `r-{16HEX}`，完全独立的会话生命周期，通过 `isolation: 'remote'` 触发

### Coordinator 编排架构

通过 `CLAUDE_CODE_COORDINATOR_MODE=1` 激活。Coordinator 不直接执行工作，通过 AgentTool 并行派发 Worker，综合结果后回复用户：

- 并行调度是核心优势，尽可能并发派发
- 通过 SendMessage 继续已有 Worker（避免重复创建）
- 通过 TaskStop 终止不再需要的 Worker

**Worker 工具过滤规则**（`agentToolUtils.ts`）：
- MCP 工具（`mcp__*`）：所有 Agent 均可用
- 交互式工具：异步 Worker 不可用
- SendMessageTool：异步 Worker 不可用
- AgentTool：子 Agent 默认不可再派生（防递归）

### 三种消息传递机制

**Task Notification**（异步 Worker 结果回传）：`<task-notification>` XML 格式注入到 Coordinator 下一轮用户消息中，含 task-id、status、summary、result、usage

**Mailbox 系统**（In-Process Teammate 通信）：基于文件系统 `~/.claude/teams/{teamName}/inboxes/{agentName}.json`，proper-lockfile 实现文件锁，Teammate 在每轮边界轮询收件箱；`from: "*"` 广播到所有 Teammate

**Pending Message Queue**（异步 Agent 继续）：`SendMessageTool` 路由到 LocalAgentTask 的 `pendingMessages` 队列，在 Agent 完成当前轮次后消费为下一轮输入

### MCP 工具动态集成

注册流程：连接 MCP Server（stdio/SSE/HTTP/WebSocket）→ ListTools RPC 发现工具 → 名称规范化（`mcp__server-name__my_tool`）→ 注入工具池传递给 Claude API

MCP 服务器连接状态：`connected` / `needs-auth` / `pending` / `failed` / `disabled`

4 种配置来源：`local`（当前目录）/ `user`（用户全局）/ `project`（项目根目录）/ `managed/enterprise`（企业推送）

### Fork 子 Agent 模式

不指定 `subagent_type` 时创建 Fork，继承父 Agent 完整上下文和系统提示词，Prompt Cache 字节完全一致（命中率高），boilerplate tag 防止 Fork 再次递归派生。

---

## 六、五大内置 Agent

内置 Agent 来自 `src/tools/AgentTool/built-in/` 目录，通过 `getBuiltInAgents()` 注册。

| Agent | 模型 | 工具权限 | 核心职责 |
|-------|------|---------|---------|
| **General Purpose** | getDefaultSubagentModel() | 全部 `['*']` | 通用研究、代码搜索、多步骤任务，兜底 Agent |
| **Explore** | haiku（外部）/ inherit（ANT） | 只读（禁 Edit/Write/AgentTool） | 极速代码库搜索，quick/medium/very thorough 三档 |
| **Plan** | inherit（主 Agent 同款） | 只读（同 Explore） | 软件架构设计，纯探索不修改，输出结构化计划 |
| **Verification** | inherit | 只读 + /tmp 写 | 对抗性验证，输出 PASS/FAIL/PARTIAL 裁决 |
| **Claude Code Guide** | haiku | 搜索 + 网络工具（无编辑类） | Claude Code / API / SDK 使用问题解答 |

**启用条件**：Explore 和 Plan 受 `BUILTIN_EXPLORE_PLAN_AGENTS` 特性 + GrowthBook A/B 实验（`tengu_amber_stoat`）控制；Verification 受 `VERIFICATION_AGENT` + GrowthBook `tengu_hive_evidence` 双重门控。

### Explore Agent

外部使用 `haiku` 模型（速度快、成本低），`omitClaudeMd: true` 省略 CLAUDE.md 加载节省 token，提示词明确要求并行工具调用。

三档彻底度（调用者在 prompt 中指定）：
- `quick`：快速扫描，适合目标明确的搜索
- `medium`：中等深度，平衡速度与完整性
- `very thorough`：彻底探索，适合架构分析

触发时机建议：3 次以上 Glob/Grep 搜索仍找不到时才调用（直接用专用工具更快）。

### Plan Agent

`model: 'inherit'`（使用主 Agent 的强模型，保证推理深度），工具集直接复用 Explore Agent 的工具集。

强制结构化输出：提示词要求响应必须以 `### Critical Files for Implementation` 结尾，列出 3-5 个关键文件路径。

**Plan Agent vs Plan Mode（权限模式）的区别**：
- Plan Agent：独立子 Agent，完整推理能力，适合复杂架构设计
- Plan Mode：主 Agent 的权限限制模式（Shift+Tab 切换），适合简单操作前快速规划

### Verification Agent

最复杂的提示词（约 130 行），`color: 'red'` + `background: true`，`criticalSystemReminder_EXPERIMENTAL` 每轮重新注入关键提醒。

**核心哲学（对抗性验证）**："Your job is not to confirm the implementation works — it's to try to break it."

明确列举 AI 验证者的两大失败模式：
1. **验证回避**：阅读代码后直接写 PASS，不实际运行
2. **被前 80% 迷惑**：看到美观 UI 或通过的测试套件就判断 PASS，忽略边界情况

按变更类型的验证策略：
- Frontend：启动 dev server + 浏览器自动化（MCP playwright）点击截图
- Backend/API：启动服务 → curl/fetch 端点 → 验证响应形状 → 测试错误处理和边界输入
- Bug Fix：**先重现原始 bug**（关键）→ 验证修复 → 回归测试
- Refactoring：现有测试 MUST pass + diff 公开 API 表面 + 行为一致性 + 性能无明显退化

强制输出格式：每个 check 必须包含真实执行的命令和实际输出，无命令运行证据的 check 判定为 SKIP 而非 PASS。PARTIAL 仅用于环境限制（工具不可用、服务器无法启动）场景。

### General Purpose Agent

唯一的全权限 Agent（`tools: ['*']`），可读写文件、运行任意 Bash、派生子 Agent。动态模型选择（`getDefaultSubagentModel()`），是跨领域任务的默认兜底选择。

### Claude Code Guide Agent

`model: haiku`，`permissionMode: dontAsk`，通过 `WebFetch` 实时获取文档（`code.claude.com/docs/`、`platform.claude.com/llms.txt`）。

会话复用机制：调用前检查是否有运行中的 `claude-code-guide` Agent，若有则通过 SendMessage 继续而非新建（保留上下文，节省初���化开销）。

动态系统提示词注入用户当前配置：已安装的自定义技能、配置的自定义 Agent、已连接 MCP 服务器列表、Plugin 命令列表、settings.json 完整内容（实现个性化建议）。

### 自定义 Agent（完全等价）

自定义 Agent 放置于 `.claude/agents/` 目录，YAML frontmatter 声明配置，与内置 Agent 完全等价，支持：
- 任意工具（包括 AgentTool 实现 Agent 嵌套）
- 最强 opus 模型
- 绑定专属 MCP 服务器
- 独立权限模式（`permissionMode`）

AgentDefinition 接口关键字段：`description`、`tools`、`disallowedTools`、`model`、`mcpServers`、`permissionMode`、`omitClaudeMd`、`hooks`、`whenToUse`

---

## 七、可借鉴的 10 个工程设计模式

### 1. 双层 Generator 架构

**问题**：会话状态管理（跨轮次记忆、费用跟踪）和行为循环（API 调用、工具执行）职责不同但紧密耦合。

**方案**：外层 QueryEngine（AsyncGenerator）管理会话生命周期；内层 queryLoop（while 循环）只关心单次请求的完整"思考-行动-观察"周期。两层通过 `yield*` 传递消息流。

**借鉴**：将"会话管理"和"行为执行"分离为两层。内层是纯状态机（输入：消息历史+配置，输出：消息流+终止原因），可独立测试；外层专注持久化、计费、SDK 适配。

### 2. 流式工具并发执行

**问题**：等待 API 响应完成后再执行工具，导致延迟串行叠加。

**方案**：StreamingToolExecutor 在 API 流式过程中，tool_use 块一旦到达立即异步开始执行。`isConcurrencySafe()` 判断并发安全性：读操作（文件读/搜索/MCP）并行执行，写操作（文件写/Bash）串行执行。可节省数百毫秒延迟。

**借鉴**：工具声明 `isConcurrencySafe: boolean`，主循环维护执行槽位。读类工具几乎总是安全的，写类工具需严格串行。

### 3. 系统提示词分段注册表

**方案**：每个提示词段由独立函数生成，通过注册表统一调用并缓存。静态段走全局 Prompt Cache，动态段每轮重新计算，BOUNDARY 标记精确分隔缓存域。

**借鉴**：将不同关注点分离到独立函数（角色/规则/环境/工具指南/记忆等），明确区分"不变内容"（全局缓存）和"会话相关内容"（动态生成）。使用注册表模式而非硬编码拼接。

### 4. 分层权限模型（免疫绕过 + 规则引擎）

**方案**：12 步决策管道，越危险的检查越靠前，部分检查"免疫绕过"。5 种权限模式满足不同使用场景。规则引擎支持内容级精细规则。

**借鉴**：区分三类规则：硬编码安全检查（永远运行）、用户可配置规则（Allow/Deny/Ask）、模式级别控制（开发/生产）。最危险操作（删除、发布、外部消息）在任何模式下都需要明确授权。

### 5. 内建三条错误恢复路径

**方案**：(1) 413 → Reactive Compact 自动摘要历史后重试；(2) max_output_tokens → 升级 max_tokens + nudge 消息最多重试 3 次；(3) 模型过载 → 自动切换 fallback 模型。三条路径相互独立，失败时才将错误暴露给用户。

**借鉴**：为每种常见 API 错误设计自动恢复策略。关键是"扣留"错误消息直到确认无法恢复，避免中间状态暴露给用户。每种恢复策略设置最大尝试次数，超限后优雅降级。

### 6. Token 预算与自动压缩机制

**方案**：每轮迭代前检查 token 数量，超阈值时调用独立压缩 API 生成摘要替换历史（AutoCompact）。TOKEN_BUDGET 特性允许指定目标 token 消耗量，不足时自动注入 nudge 消息促使 Agent 继续工作。

**借鉴**：在 Agent 主循环中定期检测上下文大小，提前触发压缩而不是等 API 报错。设计可插拔的压缩策略（滑动窗口/LLM摘要/工具结果截断）。对于长任务，提供"强制继续"机制（nudge）。

### 7. 异步 Worker + task-notification 通信

**方案**：主 Agent 通过 AgentTool 派发异步 Worker 立即返回不等待。Worker 结果以 XML 格式（task-notification）注入主 Agent 下一轮用户消息，通过 task-id 识别来源。SendMessageTool 允许向运行中的 Worker 发送追加指令。

**借鉴**：多 Agent 通信不需要复杂的实时通道。将 Worker 结果以结构化 XML/JSON 注入协调者消息流是最简单有效的方案。关键设计：每个结果带唯一 task-id（关联）、状态（检测失败）、简短摘要（快速过滤）。

### 8. MCP 工具的动态注册与名称空间化

**方案**：所有 MCP 工具自动添加 `mcp__服务器名__工具名` 前缀命名空间。服务器连接状态有明确状态机（connected/pending/failed/needs-auth/disabled）。工具发现结果被 memoize 缓存。子 Agent 可继承父 Agent 的 MCP clients 并扩展专属 MCP 服务器。

**借鉴**：设计工具生态系统时，为第三方工具使用命名空间前缀避免冲突。提供清晰的工具注册 API（工具名、描述、JSON Schema、执行函数），让第三方工具与内置工具完全等价。连接状态的可见性对调试至关重要。

### 9. 系统提示词优先级覆盖链

**方案**：4 层优先级覆盖链（overridePrompt → Coordinator → Agent 定义 → 用户自定义 → 默认），每层可"替换"或"追加"，`appendSystemPrompt` 无论哪层都追加到末尾。

**借鉴**：设计 Agent 框架时，明确定义提示词优先级层次（框架默认 → 产品定制 → 用户偏好 → 实例覆盖）。"追加"比"替换"更安全（不会意外丢失安全规则）。为高优先级提示词提供"免疫保护"机制。

### 10. Hooks 系统：工具调用前后的可编程干预

**方案**：4 类 hooks：`pre-tool-use`（可修改输入或拒绝）、`post-tool-use`（可检查输出）、`user-prompt-submit`（用户提交前）、`stop`（可阻止 Agent 继续）。以 shell 命令形式配置，通过 stdin/stdout 交互，语言无关。

**借鉴**：在 Agent 生命周期的关键节点暴露 hooks 接口（工具调用前后、LLM 响应后、循环终止前）。以进程/shell 命令实现而非代码注入，降低集成复杂度。Hook 返回值明确表达意图：allow/deny/modify。

---

## 关键源文件参考

| 机制 | 核心文件 | 关键函数/类 | 行数 |
|------|---------|------------|------|
| Agent 主循环 | `src/query.ts` | `queryLoop()` | 1729 |
| SDK 会话管理 | `src/QueryEngine.ts` | `QueryEngine.submitMessage()` | 1295 |
| 系统提示词构建 | `src/constants/prompts.ts` | `getSystemPrompt()` | 914 |
| 提示词覆盖优先级 | `src/utils/systemPrompt.ts` | `buildEffectiveSystemPrompt()` | 123 |
| 权限决策管道 | `src/utils/permissions/permissions.ts` | `hasPermissionsToUseToolInner()` | 1500+ |
| Bash 规则匹配 | `src/tools/BashTool/bashPermissions.ts` | `bashToolHasPermission()` | 1400+ |
| 并发工具执行 | `src/services/tools/toolOrchestration.ts` | `runTools(), partitionToolCalls()` | 188 |
| 流式工具执行器 | `src/services/tools/StreamingToolExecutor.ts` | `addTool(), getCompletedResults()` | 530+ |
| Coordinator 模式 | `src/coordinator/coordinatorMode.ts` | `getCoordinatorSystemPrompt()` | 300+ |
| Agent 派生执行 | `src/tools/AgentTool/runAgent.ts` | `runAgent()` | 200+ |
| MCP 客户端 | `src/services/mcp/client.ts` | `connectToServer(), listTools()` | 3348 |
| Mailbox 通信 | `src/utils/teammateMailbox.ts` | `writeToMailbox(), readMailbox()` | 200+ |

---

*基于 @anthropic-ai/claude-code v2.1.88 源码分析 · 仅供研究参考*
