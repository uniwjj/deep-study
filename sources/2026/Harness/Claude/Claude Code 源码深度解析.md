---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-harness]
---

## Claude Code 源码深度解析

**文章信息**：
- **原文标题**：Claude Code 源码深度解析
- **来源**：https://zhuanlan.zhihu.com/p/2022442135182406883
- **作者**：AeroMind（互联网行业 AI 技术专家）
- **发表时间**：2026-03-31

---

**一段话总结**：

2026 年 3 月 Claude Code 源码因 npm 包中的 .map 文件泄露，1902 个 TypeScript 源文件（512,685 行）被公开。本文从源码出发，深度拆解 Claude Code 的核心工程决策。系统以 `query.ts` 为心脏，运行 AsyncGenerator 驱动的 Agentic Loop，每轮迭代经过 Snip→微压缩→上下文折叠→自动压缩→组装请求五步预处理流水线后才调用 API，并通过 StreamingToolExecutor 实现流式工具执行。上下文压缩采用四层策略（微压缩/自动压缩/会话记忆压缩/上下文折叠），CLAUDE.md 是唯一不受压缩影响的上下文。安全体系围绕最危险的 BashTool 构建了四层纵深防御：静态安全检查（23 种模式匹配）→ 语义分析（tree-sitter AST 解析）→ AI 分类器（yoloClassifier）→ 用户确认，共 9300 行安全代码。Agent 系统有三层：内置 Agent（explore/plan/verification，通过 disallowedTools 在工具层面硬约束）、Fork 子 Agent（继承父上下文和工具集，最大化 prompt cache 命中）、Coordinator 多 Agent 协作。源码还揭示了多个隐藏功能：Capybara 模型代号（v8 虚假声明率 29-30% vs v4 的 16.7%）、Undercover 卧底模式（内部员工给外部项目提交代码时自动剥离归属信息）、KAIROS 长期运行助手模式（含 SleepTool 和 autoDream 记忆巩固）、Anti-Distillation 反蒸馏防御、以及 18 物种的 Buddy 虚拟宠物系统。作者的核心洞察：真正与 LLM 交互的核心代码仅约 6400 行，其余 50 多万行全是让 Agent 安全、高效、可靠运行的工程基础设施——"AI 是冰山一角，工程是水面之下的一切"。

---

**作者判断**：

作者的核心观点是 Agent 工程的本质不在 AI 交互本身，而在支撑其运行的庞大工程基础设施。原文："6400 行 AI 交互代码，50 万行工程基础设施。这就是 Agent Engineering 的真相——AI 是冰山一角，工程是水面之下的一切。" 作者对 Claude Code 的工程水准表达了明确的敬意——用"代码层面的对抗性设计"而非单纯 prompt 约束来保障安全，用"投机性并行预取"来优化性能，用多层压缩来与上下文窗口打"持久战"。同时作者也客观呈现了源码中的混乱（utils/ 占 180K 行、thinking blocks 处理逻辑让维护者"被坑过很多次"）和模型迭代的非线性进步（Capybara v8 某些指标反而退步）。整体态度是"以工程视角审视 Agent 系统的真实复杂度"。

---

**核心内容**：

**源码泄露背景**：2026 年 3 月 31 日，安全研究员 Chaofan Shou 发现 Anthropic 发布到 npm 的 Claude Code 包附带了 .map 文件，指向 R2 存储桶中未混淆的 TypeScript 源码。1902 个源文件，512,685 行。运行时是 Bun，终端 UI 用 React+Ink，协议层接了 MCP 和 LSP。

### 0. 整体架构 — 51 万行代码的全景图

src/ 下 35 个顶层目录，代码量分布极不均匀：

- `utils/`（180K 行）占总量三分之一，藏权限系统、bash 安全检查、模型管理、bash 解析器、文件历史、session 存储等核心逻辑
- `components/`（81K 行）是 140 多个 Ink 终端 UI 组件
- `services/`（53K 行）封装所有外部服务：Anthropic API、MCP 客户端、OAuth、上下文压缩、记忆提取、Analytics
- `tools/`（50K 行）是 40 多个 Agent 工具实现
- `commands/`（26K 行）是 50 多个用户斜杠命令

![Claude Code 整体架构图](https://pic1.zhimg.com/v2-699a0f1138b53b8dd14e58b5ccdc4398_r.jpg)

关键架构观察：
- `query.ts` 是系统心脏，所有数据流汇聚于此
- `AgentTool` 递归调用 `query.ts`，子 Agent 本质上是新的 query loop 实例，可无限嵌套
- `utils/` 不是"工具函数"，而是最复杂的业务逻辑所在地
- UI 层占 100K 行，交互复杂度接近 GUI 应用
- 核心设计原则：关注点分离——query.ts 只管循环编排，工具只管自己的逻辑，权限只管判断

### 1. Agentic Loop — 五步流水线与流式执行

核心循环是 `query.ts` 中 AsyncGenerator 驱动的 while(true) 循环。每轮迭代在调用 Claude API 前经过五步预处理流水线：

**Snip → 微压缩 → 上下文折叠 → 自动压缩 → 组装请求**

五步间有精心设计的依赖关系：Snip 裁剪的 token 数传递给 autocompact 让阈值判断更准确；上下文折叠刻意放在 autocompact 之前——如果折叠就能降到阈值以下，autocompact 就不触发，保留更细粒度的上下文。

![Agentic Loop 流程图](https://pic1.zhimg.com/v2-699a0f1138b53b8dd14e58b5ccdc4398_r.jpg)

**流式工具执行（StreamingToolExecutor）**：传统做法等模型输出完整 tool_use block 再执行工具，流式执行则在模型还在输出时就开始准备——比如模型正在输出 BashTool 参数，执行器已在做安全检查。

**Fallback 机制**：主模型流式输出失败时切换 fallback model 重试。已输出的带签名 thinking blocks 成为"孤儿消息"，用 tombstone 消息类型标记，否则后续 API 调用会报签名错误。

**用户中断处理**：打断时需为每个未返回结果的 tool_use block 补 error 类型的 tool_result，否则 API 因消息协议不完整拒绝下一次请求。

### 2. 上下文压缩 — 和上下文窗口的持久战

CLAUDE.md 在会话开始时全量加载，MCP 工具名开始时加载但完整 schema 延迟到使用时，子 Agent 完全隔离在独立上下文窗口中。

![上下文加载时序图](https://pica.zhimg.com/v2-fe013c9ee04c5bc1bac4f9543feed562_r.jpg)

四层压缩策略（从细到粗）：

1. **微压缩（microCompact）**：每次工具调用返回结果后就地缩减，对用户完全透明
2. **自动压缩（autoCompact）**：token 接近窗口时触发，阈值 = 窗口大小 - 20K（p99.99 的 compact summary 输出为 17,387 token）
3. **会话记忆压缩（sessionMemoryCompact）**：跨 compact 边界保留关键信息（如用户早期提到的关键约束）
4. **实验性策略**：响应式压缩（reactiveCompact）和上下文折叠（contextCollapse），通过 feature flag 控制

**关键架构约束**：CLAUDE.md 有四层加载层级（组织级→用户级→项目级→本地），不受压缩影响，是唯一不会被压缩吞掉的上下文。所以"把持久规则放在 CLAUDE.md 里"不是偏好，是架构约束。

### 3. Verification Agent — 代码层面的对抗性设计

Verification Agent 的设计不是靠 prompt 约束，而是在代码层面实现完整的对抗性约束：

- `background: true`：后台运行不阻塞主 Agent
- `disallowedTools` 做硬约束：在工具注册层面直接移除写操作工具（FILE_EDIT、FILE_WRITE、NOTEBOOK_EDIT 等），模型根本看不到这些工具——prompt 可被 jailbreak，工具列表不行
- `model: 'inherit'`：继承父 Agent 模型，共享 prompt cache
- `criticalSystemReminder_EXPERIMENTAL`：每轮用户 turn 重新注入的短消息，不会被压缩掉
- 触发条件：改动超过一定规模时才触发（3+ file edits、backend/API changes、infrastructure changes）
- 输出格式：精确的 `VERDICT: PASS/FAIL/PARTIAL`，给代码消费而非给人看

设计思路：**用代码约束替代 prompt 约束，用工具白名单替代行为指导，用异步执行替代阻塞等待。**

### 4. 三层 Agent 架构 — 从子 Agent 到 Agent 团队

![三层 Agent 架构图](https://pic4.zhimg.com/v2-d8f02cb9210a935d5605fa0f8a05f5e3_r.jpg)

**内置 Agent**：explore 和 plan 的 disallowedTools 包含所有写操作工具（同 Verification Agent 思路），general-purpose 做通用搜索和执行。

**Fork 子 Agent**（最有意思的中间层）：
- 继承父 Agent 完整对话上下文和 system prompt，共享 prompt cache
- `tools: ['*']` 配合 useExactTools——拿到父 Agent 完全相同的工具集，最大化 cache 命中
- 判断标准是定性的："我还需要这个输出吗？"——中间结果是否值得保留在上下文里
- 三条硬约束：不能偷看（输出写在 output_file 里）、不能换模型（model: 'inherit'）、异步通知不阻塞
- 核心主题：**信息隔离**——任何打破隔离的行为都让 fork 失去意义

**Coordinator 模式**：多个 Agent 通过 SendMessageTool 互相通信，协作完成复杂任务。

### 5. 安全架构 — 给 LLM 一个 Shell 有多危险

围绕 BashTool 构建的四层纵深防御，共 9300 行安全代码：

**第一层：静态安全检查（bashSecurity.ts）**。23 种模式匹配，防御不只是 $() 命令替换，还包括 Zsh 的 =cmd 扩展、zmodload 危险模块、Unicode 零宽空格伪装、IFS 修改等。设计哲学：宁可误杀，不可漏放。一个正则少了几个字符就是安全漏洞（`> /dev/nullo` 匹配 `/dev/null` 前缀）。

**第二层：语义分析（bashPermissions.ts）**。用 tree-sitter 解析命令 AST，理解实际意图。pathValidation.ts 维护命令-操作类型映射表，readOnlyValidation.ts 维护安全命令白名单（如 fd 命令的 -x/--exec 被刻意排除）。

**第三层：AI 分类器（yoloClassifier.ts）**。Auto mode 的核心，独立 LLM 调用判断命令安全性，有自己的 system prompt 和权限模板，区分内部/外部用户权限策略。结果是结构化 JSON（Zod 校验），超时 fallback 为 ask。

**第四层：用户确认**。前三层都通过且不在白名单内才弹窗，用户选择可形成 alwaysAllow/alwaysDeny 规则。

**投机性分类器检查**：模型还在流式输出 BashTool 参数时，系统已并行运行分类器——把串行等待变成并行预取。

### 6. autoDream — 记忆系统的隐藏深度

记忆系统三层：CLAUDE.md（用户手写持久指令）、auto memory（自动提取的学习记录）、隐藏机制。

- MEMORY.md 双重截断：200 行和 25KB（有人在 200 行内塞了 197KB）
- 记忆提取是后台 Agent（主对话的 fork），严格工具限制，turn 数有预算优化
- 约束：只能用最近对话内容更新记忆，不能 grep 源文件验证
- `src/services/autoDream/`：consolidation prompt 和 lock，后台整理巩固记忆
- `src/services/teamMemorySync/`：团队记忆同步，含 secretScanner.ts 扫描敏感信息

方向：**记忆不只是"存下来"，还需要整理、巩固、共享、脱敏。**

### 7. 隐藏彩蛋

**Capybara — 泄露的模型代号**：v8 虚假声明率 29-30% vs v4 的 16.7%，说明模型迭代不总是线性进步。用 String.fromCharCode 编码物种名绕过自己的泄露检测。

**Undercover Mode — 卧底模式**：内部员工给外部开源项目提交代码时自动激活——剥离归属信息、隐藏模型身份、拦截内部信息。默认开启，无法确认是内部 repo 就保持卧底状态。

**Tengu — 内部项目代号**：出现 1498 次，所有 analytics event 以 tengu_ 为前缀。

**KAIROS — 长期运行助手模式**：关联 SleepTool（休眠等待周期性唤醒）、autoDream 记忆巩固、proactive 主动模式。Agent 不再"问一次答一次"，而是持续运行、主动观察、定期整理记忆。SleepTool prompt："Each wake-up costs an API call, but the prompt cache expires after 5 minutes of inactivity — balance accordingly."

**Anti-Distillation — 反蒸馏防御**：API 请求带 fake_tools 参数，注入虚假工具调用模式，让蒸馏模型学到错误行为。

**Buddy System — 虚拟宠物**：18 物种、6 种眼睛、8 种帽子、5 个稀有度等级（common 到 legendary，权重 60:25:10:4:1），5 项属性（DEBUGGING/PATIENCE/CHAOS/WISDOM/SNARK），名字和性格由模型生成。

### 8. 冰山之下

真正与 LLM 交互的核心代码（query.ts、QueryEngine.ts、claude.ts）约 6400 行。剩下 50 多万行是让 Agent 安全、高效、可靠运行的工程基础设施。

**核心洞察：6400 行 AI 交互代码，50 万行工程基础设施。Agent Engineering 的真相——AI 是冰山一角，工程是水面之下的一切。**
