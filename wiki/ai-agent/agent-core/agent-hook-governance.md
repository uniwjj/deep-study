---
title: Agent Hook 治理——用 Hook 切面堵住 LLM 的偷懒、越权与失忆
description: DECO 数仓 Agent 引擎的 Hook 护栏体系：通过 beforeTool/afterTool 切面实现长文本读写两侧 offload、HITL 危险操作门禁、Hook→state→Attachment 上下文联动闭环，在框架层确定性兜底 prompt 管不住的 LLM 行为缺陷
aliases: [agent-hook-governance, Agent Hook Governance, Hook 治理, Hook Guard, Agent 护栏]
tags: [ai-agent, concept, practice]
sources: [2026/07/16/Agent 治理：用 Hook 堵住 LLM 的偷懒、越权与失忆.html]
created: 2026-07-16
updated: 2026-07-16
---

# Agent Hook 治理

> 核心主张：**prompt 管不住的，框架来堵。** LLM 的"偷懒"（长文本截断/略写）、"越权"（未确认发布/回刷）和"失忆"（改表不查风险、产出不知汇报）不是 prompt engineering 能解决的问题——唯一解法是在 Agent 框架层，让偷懒和越权的路径**代码级强制走不通**，让失忆的已知盲区**确定性补齐**。

## 问题根因

三种典型 LLM 行为缺陷，根因各不相同：

| 问题 | 表现 | 根因 | 为什么 prompt 管不住 |
|------|------|------|---------------------|
| **偷懒** | 长 SQL 截断、占位略写（`-- 其他字段...`）、跳步骤 | 长脚本**物理上超出 token 预算** | prompt 改不了模型输出时 token 耗尽的事实 |
| **越权** | 在方案设计阶段未确认就调发布工具 | 模型无法区分"查询"和"发布"的**可逆性差异** | prompt 是软约束，模型可能绕过 |
| **失忆** | 改表不分析下游影响、产出图表不知告知用户 | 模型追求**最短完成路径**，额外 tool call = 多一步推理 | 信息不对称——LLM 不知道 Python 脚本产出了什么文件，不是"忘了查"而是"根本不知道有东西该查" |

## Hook 链架构

拦截这三类问题的切入口，是 Agent 框架普遍提供的 **Hook（Callback）机制**：框架在 Agent 运行的每个关键节点暴露出切面，拦截逻辑挂载到切面上——到点框架自动回调，同一切面可挂多个、按序执行。

### 核心切面

| 切面 | 触发时机 | 治理用途 |
|------|---------|---------|
| **beforeTool** | 工具真正执行前，可改入参、可直接拦截 | 长脚本回写前从文件加载全文（Onload）、危险操作确认（HITL 门禁） |
| **afterTool** | 工具执行后、结果回给 LLM 前，可改返回值 | 长脚本拉取后把内容替换成引用句柄（Offload） |
| beforeModel / afterModel | 每次请求 LLM 前/后 | 响应用户取消等 |
| beforeAgent / afterAgent | 单个 Agent 运行前/后 | 对话持久化等 |

### 设计原则

**基础设施和推理逻辑解耦**——Hook 切面上的逻辑独立运作，模型的 ReAct 循环不用感知；新增/删除一个 Hook，主流程一行代码都不用改。

## 三大护栏机制

### 一、长文本完整性护栏——治"偷懒"

#### 问题定位

长产物的偷懒是**结构性问题**。无论修改场景（存量长 SQL 局部改写）还是新建场景（LLM 生成目标态长 SQL），都要经过拉取（US 平台 → LLM）和写回（LLM → US 平台）两段：

| 阶段 | 偷懒形式 | 后果 |
|------|---------|------|
| 拉取 | 流式 token 自截断 | 文件截断成残缺 SQL，回写即生产事故 |
| 拉取 | view→create_file 复印重写 | 输出 token 翻倍，自截断概率近 100% |
| 拉取 | 占位/略写（`(SQL略)`、`-- 其他字段...`） | 落盘脚本不可执行 |
| 写回 | scriptContent 入参自截断（长 SQL 在 JSON 串里被截断） | 提交残缺脚本，平台无校验 |

#### 根治方案：读写两侧 offload + 引用句柄

核心思路：**LLM 永远不直接接触脚本全文。** 长内容全文留在沙箱文件，LLM 上下文里只有一句引用句柄，LLM 只用 `str_replace` 小步改写。

##### 读侧：Offload Hook（afterTool）

工具拉取到含 `scriptContent` 的响应 → Hook 拦截 → 全文写入沙箱只读快照 → 响应替换为引用句柄：

```
<offloaded to /sandbox/{taskName}.remote.etl (read-only snapshot, length=37814 chars).
 To start editing, run copy_file(...) first, then str_replace.>
```

**关键设计点**：
- **响应形态适配**：单条 Map 和数组都支持；数组下每条 item 独立判定，任一落盘失败仅该条降级
- **失败降级而非阻断**：落盘失败 → 该条返回原 `scriptContent`，让 LLM 至少拿到内容（承担自截断风险），不阻断主流程

##### 写侧：Onload Hook（beforeTool）

工具声明 `scriptContent` 和 `scriptFilePath` 两个互补参数（**框架契约**）：
- `scriptFilePath` 是纯框架参数，Onload Hook 从沙箱读全文覆盖 `scriptContent`，然后剥离 `scriptFilePath` 再转发给工具
- 下游实现侧不感知 `scriptFilePath`，框架可独立演化
- `scriptFilePath` 不在白名单路径/文件不存在/内容为空 → **抛异常阻断**（写侧与读侧不同：读侧降级，写侧必须阻断）

##### 表侧对称 offload

宽表 200–500 列场景，表侧也挂了完全对称的 Hook——DDL 正文 + columns 都走 offload，心智模型与任务侧一致。

#### 多重防线（对应数仓四阶段）

整个护栏体系贯穿数仓开发流水线的各阶段：设计→拆解→执行→验证。每一阶段的防线对应流水线的一个物理兜底点，Onload 阻断（红线）防止残缺脚本进入后续环节，Offload 降级（橙线）保证读侧不阻塞主流程。

#### 效果量化

| 维度 | 治理前 | 治理后 |
|------|-------|-------|
| offload 策略 | SQL 原样进出上下文 | **全量落盘**，所有 scriptContent 自动换引用句柄 |
| 修改任务工具调用输出 token | 每轮重传整段 SQL | **直降约 90%** |
| SQL 复印自截断 | view→重写路径下概率近 100% | **物理消除**（只走 str_replace 小步改） |
| 读侧失败处理 | — | 降级透传，可重试 |
| 写侧失败处理 | — | 阻断工具调用 |

### 二、HITL 危险操作门禁——封"越权"

#### 核心原则

**prompt 是软约束，不是安全边界。** 任何不可逆操作（发布、回刷、冻结/解冻、终止）必须在框架层有代码级强制确认：没拿到用户明确授权，工具物理上执行不了。

#### 实现：配置驱动的 beforeTool 守卫

危险工具守卫挂在 `beforeTool` 切面上，危险工具清单是 **YAML 配置驱动**的：

```yaml
deco:
  dangerous-tools:
    - name: packCommit
      required-state: confirm_pack        # 授权标记
      hint: "需要用户先选择发布方式"
      confirmation:
        title: "请确认发布方式"
        options:
          - {id: direct, label: "直接发布（免审批）", value: direct}
          - {id: approval, label: "提交审批", value: approval, hasInput: true, inputPlaceholder: "请输入审批人RTX"}
          - {id: draft, label: "保存草稿", value: draft}
          - {id: edit_more, label: "我再改改", value: edit_more}
```

实际配置了多个危险工具，每个对应不同 `requiredState` key：`confirm_pack`（发布提交）、`confirm_deploy`（触发发布）、`confirm_upsert_datasource`（数据源变更）、`confirm_transfer_task_upsert`（同步任务变更）。

#### 确认流程：拦截 → 弹框 → 用户选择 → 放行

1. Agent 调用危险工具 → beforeTool Hook 检查 `session.state` 中是否存在对应 `requiredState` key
2. 不存在 → 抛 `INTERACTION_BOX` 事件（SSE 流式协议的 `CUSTOM` 事件子类型），前端渲染确认框
3. 阻断工具调用，Agent 暂停
4. 用户选择后，前端通过 REST API 把选择写入 `session.state`，发起续跑请求
5. LLM 重调该工具时 state 已就位，守卫放行

确认框支持**带输入控件的选项**（`hasInput` / `inputPlaceholder` / `inputType`），承载"填审批人""填回刷日期"等结构化表单——不只是 yes/no。

#### 行业对比

| 框架 | 开箱程度 | 交互模式 | DECO 差异 |
|------|---------|---------|----------|
| **ADK** `ToolConfirmation` | ✅ 一行配置 yes/no；高级确认 `requestConfirmation()` 可带 payload | yes/no + payload，无原生多选 UI | DECO 需要多选项 + 输入控件 + 变更预览 |
| **LangGraph** HITL Middleware | ✅ `interruptOn` 声明式配置 | approve/edit/reject/respond | DECO 需要业务级集成确认 |
| **Claude Code** `PreToolUse` | ✅ shell 脚本 + `permissionDecision` | deny/allow/ask | DECO 需要多选项带参数 |
| **DECO** `DangerousToolGuard` | ❌ 自研 | 多选项 + 带输入控件 + 变更预览 `COMMIT_PREVIEW` | 业务级：发布前展示变更清单、确认框带参数、危险清单配置驱动、与 SSE 流式协议一体 |

**结论**：ADK 原生 `ToolConfirmation` 已可覆盖简单 yes/no 场景，且框架处理暂停/恢复和防循环——恰是自研最易出 bug 处。DECO 自研的必要性在于"框架的 HITL 不够业务化"——需要变更预览、结构化确认表单、配置驱动危险清单、SSE 协议集成。

### 三、上下文联动闭环——补"失忆"

#### 问题本质

LLM「该做但没做的」——这是第三类更隐蔽的问题：

- 改了 DDL 字段重命名 → 不主动分析下游哪些表受影响
- Python 脚本产出图表 → 不主动告知用户图在哪
- 表结构变更 → 不回查刷新过时的元数据

根因：**主动探测 = 额外一次 tool call = 多耗 token。** 模型天然追求以最少 token 完成任务，不会主动给自己加检查步骤。除此之外还有**信息不对称**——LLM 不知道 Python 脚本产出了什么文件，不是"忘了查"，而是"根本不知道有东西该查"。

#### 范式：Hook 采集事实 → 写 state → Attachment 注入下一轮 prompt

解耦为两段：

- **Hook（afterTool）**：采集副作用事实，写入 `session.state`——**确定性触发**，不依赖 LLM 自觉
- **Attachment（下一轮 prompt 注入）**：从 state 读取，拼入下一轮 system prompt——**时机正确**，不污染当前轮上下文

对比让 LLM 自己"记得去查"的方案：

| 方案 | 可靠性 | token 开销 | 偷懒风险 |
|------|-------|-----------|---------|
| prompt 写"改表后记得分析风险" | ❌ 软约束 | 无额外 | ✅ 高 |
| 单独发一轮"请分析风险" | 🟡 依赖调度 | 额外一轮 | ✅ 中 |
| **Hook → state → Attachment** | ✅ 确定触发 | 无额外（结果复用） | ❌ 零 |

#### 案例一：RiskAnalysisHook——改表后自动注入风险分析

`afterTool` 上挂 `RiskAnalysisHook`：
- 判定"改表"语义：带 `tableId` 参数的 `upsertTable` 才是改表（新建表不带 `tableId`，直接跳过）
- 调血缘 API 分析下游依赖 → 风险结论写 state
- 下一轮 Attachment 注入，LLM 自然输出类似：

> ⚠️ 刚刚修改了 dws_order_detail 表，下游影响：dws_channel_report (HIGH) — 依赖 order_amount；ads_daily_summary (MEDIUM) — 依赖 order_status。建议检查这两张表 ETL。

**注**：此处风险分析为事后下游影响评估，非改表前拦截——事前拦截由 [[#二、HITL 危险操作门禁——封 越权|HITL Guard]] 负责。

#### 案例二：PythonImageHook——自动发现并呈现生成产物

`beforeTool` 加文件快照 → `afterTool` 对比 → 发现新 `.png`/`.jpg`/`.svg` → 写 state → Attachment 注入预签名 URL → **前端直接内联渲染图片**。

关键设计点：
- **前后快照对比**比让 LLM 用 `bash ls` 查文件可靠得多
- **只关心图片**——不处理脚本和数据文件，避免 state 塞无关文件列表
- **累积写入**——一次 Python 执行产出多张图，全部累积，一次注入

#### 行业定位

| 框架/工具 | 机制 | DECO 差异 |
|----------|------|----------|
| ADK `ArtifactService` | 存储→LLM 主动 load | DECO 是**事件驱动 + 自动注入**，不依赖 LLM 主动 load |
| LangGraph checkpointer | 流程控制 state | DECO state 更轻量，专用于"事实采集→上下文注入" |
| Claude Code `SessionStart`/`UserPromptSubmit` | 会话级 hook | DECO 在**工具调用后、下一轮 LLM 调用前**注入，时效性更强 |
| CrewAI `TaskMemory` | 跨 Agent 协调 | DECO 单 Agent 内多轮 |

DECO 这套机制的特殊之处：不是「存储→读取」的被动模式，而是**「事件→采集→注入」的主动流水线**。Hook 不等着 LLM 来查 state，而是主动把结论 push 进下一轮 prompt——即便 LLM 完全不知道 state 里有风险分析结果，Attachment 也会让它「看到」。

## Hook 全景生态

同一套 Hook 链上，DECO 实际挂了**十余个 Hook**，覆盖全部横切关注点：

| 分类 | Hook | 挂载点 | 职责 |
|------|------|-------|------|
| **长文本护栏** | TaskScriptOffloadService / OnloadService | afterTool / beforeTool | ETL 脚本读写两侧全量 offload |
| | TableColumnsOffloadService / DdlColumnsOnloadService | afterTool / beforeTool | 宽表 columns 读写两侧 offload |
| | DdlBodyOffloadService | afterTool | DDL 正文无条件落盘 + 表元数据自动拼接 |
| **危险操作护栏** | DangerousToolGuard | beforeTool | 危险工具拦截 + HITL 确认 |
| **工具返回处理** | LineageResponseOffloadService | afterTool | 血缘原始响应写 offload |
| | ToolResponseTruncator | afterTool | 超大返回智能裁剪（超阈值触发 Rerank 重排） |
| | ToolResponseFormatter | afterTool | 工具返回结构化格式化 |
| **可观测 & 持久化** | ToolCallLogHook | before/afterTool | 异步记录工具入参/出参/耗时/成功率 |
| | LoggingHook | 多点 | Agent 执行链路日志 |
| | ConversationPersistenceHook | before/afterAgent | 对话落盘持久化 |
| **前端联动** | FileEventHook | onRunEvent | 沙箱文件事件实时推前端 |
| | HookEventHook | onRunEvent | 通用事件通知前端刷新 |
| **上下文联动** | RiskAnalysisHook | afterTool | 改表后触发下游风险分析 → state → Attachment |
| | PythonImageHook | before/afterTool | Python 产物图片自动发现 → state → Attachment |
| **沙箱环境** | SandboxEnvHook | beforeTool | 工具调用前后确保沙箱一致性 |

所有 Hook 遵循同一原则：**不改业务循环、不动工具实现，把横切逻辑挂在切面上。**

## 行业对比总结

### 长文本 offload

| 工程 | 读侧策略 | 写侧策略 | DECO 差异 |
|------|---------|---------|----------|
| ADK `ArtifactService` | `save_artifact`/`load_artifact` API | ❌ 需工具内手动调 API | 非 Hook 层自动拦截 |
| LangGraph DeepAgents | 中间件：>20k token 自动落盘 | ❌ 只做读侧 | 无数仓"写长 SQL 回写"场景 |
| Claude Code | Read 分页、Edit 强制 str_replace | ❌ 无 offload | DECO 需要**两端对称 offload** |

### 关键结论

ADK 和 LangGraph 都有内置 offload 能力，但只做到读侧——因为大部分 Agent 场景不需要把长产物原样发回外部 API。DECO 数仓场景的不同在于需要**两端对称 offload**，且写回需额外加固（`scriptFilePath` 框架协议、只读快照/工作副本分离、注释块字段名识别、列级 offload、失败语义按代价差异化）。

## 相关页面

- [[agent-architecture-patterns]] — Agent 10 大设计模式，Hook 优于插件的架构原则
- [[ai-governance]] — AI 治理框架，Hook 治理是框架层治理的具体实现
- [[data-agent-enterprise-practice]] — 蚂蚁 Data Agent 企业实践，DECO 是另一条企业级实践路径
- [[agent-harness-overview]] — Harness 六承重层，Hook 对应横切关注点层
- [[agent-tdd-workflow]] — Agent 开发中的 TDD 流程，Hook 测试是其中一环
- [[agent-mcp-protocol]] — MCP 工具协议，Hook 是对工具调用的切面增强
