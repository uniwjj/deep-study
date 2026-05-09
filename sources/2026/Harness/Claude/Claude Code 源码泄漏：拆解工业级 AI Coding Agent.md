---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-harness]
---

## Claude Code 源码泄漏：拆解一个工业级 AI Coding Agent 到底是怎么造出来的

**文章信息**：
- **原文标题**：Claude Code 源码泄漏：拆解一个工业级 AI Coding Agent 到底是怎么造出来的
- **来源**：https://juejin.cn/post/7623066322222153728
- **作者**：AI程序员
- **发表时间**：2026-03-31

---

**一段话总结**：

本文从泄露的 Claude Code 全量源码出发，从启动流程、Agent 循环、工具系统、权限模型、上下文管理、多 Agent 协作等 17 个维度逐一拆解这个日活百万的工业级 AI Coding Agent。Claude Code 是用 TypeScript+React(Ink) 构建的、运行在终端里的完整 Agent 操作系统，约 60+ 内置工具、60+ 用户命令。核心循环是 QueryEngine 驱动的 ReAct 循环，支持流式工具执行（StreamingToolExecutor）和智能并行/串行分区。权限系统是最重的部分之一，8 层 Bash 安全纵深防御（23 个独立检查 ID）+ 多阶段权限检查（deny rules→ask rules→工具自检→permission mode→白名单→classifier 并发竞速）。多 Agent 架构包含 Coordinator 模式和 Agent Swarms，Fork Subagent 通过 prompt cache 共享实现低成本后台执行。记忆系统是三层渐进式管线（Auto Memory Extraction→Session Memory→Auto Dream 跨会话整合）。文章还揭示了两套 Feature Flag 机制（构建期死代码消除 vs 运行期远程配置）、KAIROS/Assistant/Bridge 等多种隐藏产品形态、以及 36 行极简 Store 状态管理等工程模式。作者核心判断：这不是 demo 级产品，而是经过深度工程化的工业级 Agent 系统，是目前最值得学习的 Coding Agent 参考实现。

---

**作者判断**：

作者明确判断 Claude Code 是"经过深度工程化的工业级 Agent 系统"，原文："Claude Code 不是一个 demo 级产品，它是一个经过深度工程化的工业级 Agent 系统。从启动的并行初始化优化到工具并发调度，从多层权限模型到编译期死代码消除，每一个细节都透露着'这是一个真实服务百万用户的产品'。" 作者认为值得 Agent 开发者学习的四个设计：权限系统是架构层面的第一等公民、工具并行/串行自动判断是被低估的优化、Feature Flag+编译期消除让一套代码支撑多种产品形态、Agent 是可嵌套组合协作的执行单元。同时作者也指出七个局限：上下文窗口硬约束、权限系统维护成本高、循环依赖问题、Feature Flag 膨胀、Bun 锁定、Forked Agent 隐性 API 成本、native-ts 功能子集。整体态度是"推崇设计、正视局限"。

---

**核心内容**：

**关键判断**：
- 技术栈：Bun 运行时（构建期依赖 `bun:bundle` 做死代码消除），部分路径兼容 Node，UI 用 React Ink，API 用 Anthropic SDK，约 60+ 内置工具、60+ 用户命令
- 用纯 TypeScript 重写了三个 native 依赖（syntax-highlighted diff、fuzzy file search、yoga flexbox layout），但仍有 audio-capture-napi、url-handler-napi、image-processor-napi、modifiers-napi 等 NAPI 模块在用
- 核心循环：经典 ReAct + 流式执行 + 并行工具调用 + 自动上下文压缩
- 权限模型：多阶段防线，安全性设计扎实
- 多 Agent：Coordinator 模式 + Agent Swarms，已不是单 Agent 产品
- Bridge 模式：WebSocket 实现桌面↔远程服务器会话桥接
- 记忆：三层渐进式管线（Auto Memory→Session Memory→Auto Dream）
- 扩展体系：Plugin+Skill+Hooks+MCP Skill Builders，已是完整扩展平台

### 二、启动流程：极致的并行初始化

把昂贵的 I/O 操作（钥匙串读取、MDM 配置读取）提前到 import 阶段并行执行。Node/Bun 的 import 本身需要 ~135ms 加载模块，这段时间正好用来做异步预热。

启动序列：main.tsx → MDM Reader（并行）+ Keychain（并行）→ Trust Dialog（安全门，用户确认信任前不加载完整环境变量和配置）→ GrowthBook reset/reinit → applyConfigEnvironmentVariables → initializeTelemetryAfterTrust → launchRepl。

关键点：
- `-p`/`--print` 非交互模式下 trust 是隐式的
- 两套 Feature Flag 不要混淆：`feature()` 来自 `bun:bundle` 是构建期死代码消除；GrowthBook/Statsig 是运行期远程配置
- 大量使用 `require()` 延迟加载打破循环依赖

### 三、Agent 核心循环：QueryEngine 是大脑

两条路径：交互模式 REPL→query()（直接调用）；Headless/SDK 模式 QueryEngine→query()。**REPL 并不经过 QueryEngine**，QueryEngine 注释明确写了"供 headless/SDK 使用"。

**QueryEngine** 是 headless/SDK 模式下每个对话的状态持有者，持有消息历史、中断控制、权限拒绝记录、累计 token 用量、文件状态缓存、技能发现追踪。

**query()** 函数是真正的执行引擎，generator 模式的 Agent 循环。四个工程亮点：

1. **流式工具执行（StreamingToolExecutor）**：不等 LLM 完整输出后执行工具，流式响应过程中就开始准备工具调用

2. **工具并行 vs 串行智能分区**：两套实现——旧路径 toolOrchestration.ts（批量分区，连续的 concurrency-safe 工具并行执行最多 10 并发）；新路径 StreamingToolExecutor（流式并发，一边接收 tool_use 块一边立即执行）。并发安全性由每个工具自己的 `isConcurrencySafe(input)` 方法根据具体输入参数判断。错误传播通过 siblingAbortController 取消兄弟进程但不中断父 query 循环

3. **自动上下文压缩**：阈值 = 有效上下文窗口 - 13000 tokens 缓冲。至少六种压缩路径：Auto Compact（摘要压缩）、Session Memory Compaction（重要信息沉淀）、Reactive Compact（prompt_too_long 紧急压缩）、Microcompact（增量压缩）、History Snip（按边界截断）、Context Collapse（折叠大块上下文）。执行顺序：applyToolResultBudget → snipCompactIfNeeded → microcompact → contextCollapse → autocompact

4. **max_output_tokens 恢复循环**：回复被截断时最多自动重试 3 次

### 四、工具系统：60+ 工具的注册与分发

内置工具注册表 tools.ts 是注册中心，MCP 工具在 MCP client 层动态创建，最后由 assembleToolPool() 合并进统一工具池。

工具分类：文件操作（FileRead/Edit/Write/NotebookEdit）、搜索（Glob/Grep/WebSearch）、执行（Bash/PowerShell）、Agent（AgentTool/TeamCreate/Delete/SendMessage）、任务（TaskCreate/Get/Update/List/Stop/Output）、计划（EnterPlanMode/ExitPlanMode/VerifyPlanExecution）、MCP 资源、其他（WebFetch/Todo/Skill/AskUserQuestion/LSP/Brief）、高级（ToolSearch/Workflow/Cron/Monitor/Sleep）、Worktree、Kairos/Assistant、调试。

**Feature Flag 驱动的条件编译**：使用 Bun 的 `feature()` 做编译期死代码消除，flag 关闭时对应 require() 不执行，代码被 bundler 彻底剔除。

**工具权限过滤**：deny rules 过滤，被 deny 的工具连 schema 都不发送给模型——模型根本不知道这些工具存在。

### 五、权限系统：多阶段防线

多阶段权限检查流程：Deny Rules（全局黑名单直接拒绝）→ Ask Rules（强制询问用户确认）→ tool.checkPermissions / safetyCheck（工具自身权限检查）→ Permission Mode（自动决策层）→ Always Allow Rules（白名单匹配）→ Hooks/Classifier（与用户确认对话框并发竞速，哪个先返回用哪个）。

权限模式比"三种"多得多：acceptEdits（自动接受编辑）、bypassPermissions（旁路全部自动通过）、default（敏感操作需确认）、dontAsk（不弹确认框，后台 agent 用）、plan（只读自动通过）。内部还有 auto 和 bubble（权限冒泡：子 agent→父 agent）。

Bash 工具安全检查特别复杂：AST 解析（parseForSecurity）、sed 编辑检测（sedEditParser）、破坏性命令警告（destructiveCommandWarning）、沙箱执行（shouldUseSandbox）、路径验证（pathValidation）。

### 六、任务系统：7 种任务类型

TaskType：local_bash、local_agent、remote_agent、in_process_teammate（Agent Swarms）、local_workflow、monitor_mcp、dream（后台记忆整合）。每个任务有密码学安全的 ID（前缀 + 8 字节随机，36^8 ≈ 2.8 万亿种组合）。

### 七、多 Agent 协作

**Coordinator Mode**：CLAUDE_CODE_COORDINATOR_MODE=1 时，协调者只能用 AgentTool/TaskStopTool/SendMessageTool/SyntheticOutputTool（不能直接操作文件），工作者才能用 Bash/Read/Edit。

**Agent Swarms**：TeamCreateTool/TeamDeleteTool/SendMessageTool/ListPeersTool，直接内建在工具系统里而非外部编排框架。

**子 Agent 执行流程**：创建独立 FileStateCache → 编译专属 System Prompt → 连接 MCP 服务器 → 启动独立 query() 循环 → 记录侧链 transcript。外部构建中 AgentTool 默认禁止（防无限递归），内部构建允许嵌套。

**Fork Subagent**（FORK_SUBAGENT feature flag）：继承完整上下文 + prompt cache 共享（system prompt 字节完全一致）+ 权限冒泡（'bubble' 模式）+ 防递归（isInForkChild() 检测）。本质是"用当前对话上下文开分支做事"，而非"创建新 agent 从零开始"。

### 八、上下文系统

System Prompt 组装：基础 System Prompt（prompts.ts）+ 环境信息 + systemContext（git 状态快照）+ userContext（CLAUDE.md）+ Memory 文件 + Coordinator 上下文 + customSystemPrompt/appendSystemPrompt + Skills 上下文。

**CLAUDE.md 多层级配置**：加载顺序从低到高——Managed memory（/etc/claude-code/CLAUDE.md）→ User memory（~/.claude/CLAUDE.md）→ Project memory（CLAUDE.md + .claude/CLAUDE.md + .claude/rules/*.md，从 git root 到当前目录递归查找）→ Local memory（CLAUDE.local.md）。支持 @include 指令引用其他文件。

**Memory 三层渐进式管线**：

1. **Auto Memory Extraction**：每轮 query 结束后，后台 forked agent 自动提取关键洞察写入项目记忆目录。使用 runForkedAgent() + Prompt Cache 共享，有 cursor 追踪避免重复，工具受限
2. **Session Memory**：三重阈值触发（10K tokens 初始化 + 5K tokens 增量 + 3 次工具调用），可替代 autocompact 的摘要（增量提取质量可能更好）
3. **Auto Dream**：距离上次整合 ≥ 24h + 积累 ≥ 5 个 session transcript 时启动 dream 任务，forked agent 遍历多个 session transcript 整合记忆。有完整生命周期管理，可被 kill 并回滚

MEMORY.md 入口文件：200 行 + 25KB 双重截断保护（实际观测到 197KB 在 200 行以下的情况）。

### 九、MCP 集成

四种传输协议：Stdio、SSE、StreamableHTTP、WebSocket。内建 OAuth 流程支持 MCP 服务器认证。prefetchAllMcpResources() 预加载资源，prefetchOfficialMcpUrls() 从官方注册表获取服务器列表。MCP 工具和内置工具在 assembleToolPool() 中统一合并，内置工具优先。

### 十、Bridge 模式：远程执行架构

桌面 VS Code 连接远程服务器，权限弹窗在本地桌面显示，实际工具执行在远程服务器，通过 WebSocket 多路复用传输。支持 JWT 认证和设备信任。

### 十一、扩展体系：Plugin + Skill + Hooks

**Plugin**：Built-in Plugins（随 CLI 发布）+ Marketplace Plugins（声明在 settings，缓存在 ~/.claude/marketplaces/，后台安装不阻塞启动）。PluginInstallationManager 实现完整 diff-reconcile 流程。

**Skill**：带 YAML frontmatter 的 Markdown 文件，支持 @include/argument 替换/shell 执行。来源：bundled/user/project/plugin/MCP。MCP Skill Builders 可动态注册。

**Hooks**：四种类型——command（shell，10min 超时）、prompt（LLM prompt，30s）、agent（多轮 Agent，60s）、http（HTTP POST，10min）。支持进度增量输出。

### 十二、UI 层

用 Ink（React for CLI）构建。完整 Vim 模式（normal/insert/visual/command）、语音输入/输出（要求 Anthropic OAuth）、虚拟滚动消息列表、Buddy 虚拟宠物系统（FNV-1a 哈希 + Mulberry32 PRNG 确定性生成，5 级稀有度）。

**native-ts 纯 TS 重写**：color-diff/（替代 Rust syntect）、file-index/（替代 Rust nucleo）、yoga-layout/（替代 Meta C++ yoga-layout），显著缩小 NAPI 依赖面。

### 十三、Bash 安全：8 层纵深防御

23 个独立安全检查 ID，归纳为 8 层：Quote Extraction（三路并行解析引号）→ Dangerous Patterns（命令替换/进程替换/zsh equals expansion/heredoc）→ Zsh-Specific Threats（zmodload/emulate/zf_rm）→ Path Validation（每命令独立提取路径）→ Advanced Threats（IFS 注入/大括号展开/控制字符/Unicode 空白/嵌入换行）→ Destructive Warning（rm -rf/git reset --hard/DROP TABLE 等展示警告但不阻止）→ Sed Edit Parser（解析 sed -i 为文件编辑操作）→ Permission Framework（只读命令自动放行/权限树匹配）。

### 十四、隐藏产品形态

从 feature flag 和代码结构推测：CLI（默认构建）、KAIROS/Assistant（PushNotification/SendUserFile/SubscribePR/session chooser/install wizard）、Coordinator（多 Agent 协调者）、Bridge/Remote（远程会话桥接）。一套代码库同时开发本地 CLI、远程托管和可能的 SaaS assistant 产品。

### 十五、值得注意的工程模式

1. **两套 Feature Flag**：构建期（bun:bundle feature()，维度是产品形态）vs 运行期（GrowthBook/Statsig，维度是灰度发布和 A/B 测试）
2. **懒加载打破循环依赖**：大量 require() 延迟加载
3. **Memoize 无处不在**：git 状态等昂贵操作被 memoize
4. **内部 vs 外部用户**：内部员工有额外工具（REPLTool/ConfigTool/TungstenTool/SuggestBackgroundPRTool）
5. **Forked Agent 模式**：Prompt Cache 共享的后台执行，system prompt 字节完全一致确保 cache 命中，GrowthBook 冷/热启动不同值时通过 renderedSystemPrompt 显式传递
6. **36 行极简 Store**：state/store.ts 只有 36 行，通过 useSyncExternalStore 集成到 Ink，没有 Redux/Zustand

### 十六、局限与坑

上下文窗口硬约束（压缩必然丢失信息，连续失败 3 次放弃）、权限系统维护成本高、循环依赖问题（大量 lazy require）、Feature Flag 膨胀、Bun 锁定（深度使用 bun:bundle）、Forked Agent 隐性 API 成本（每个后台 forked agent 都是一次 API 调用）、native-ts 功能子集（极端场景可能表现不一致）。

### 十七、核心结论

**Claude Code 不是 demo 级产品，是经过深度工程化的工业级 Agent 系统，是目前最值得学习的 Coding Agent 参考实现。**

四个值得 Agent 开发者学习的设计：
1. 权限系统不是事后补丁，而是架构层面的第一等公民
2. 工具并行 vs 串行的自动判断是被低估但影响很大的优化
3. Feature Flag + 编译期消除让一套代码支撑多种产品形态
4. Agent 不是单体，而是可嵌套、组合、协作的执行单元
