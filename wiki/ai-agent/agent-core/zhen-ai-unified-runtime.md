---
title: 真机智能统一运行时主链（C2AI2X 生态架构评审）
description: 北京真机智能董事长兼首席科学家刘智勇在 2026 Agent 大会的架构评审——从 C2AI2X 协议切入，把真机生态里前台（ZhenIns）、平台（platform-core）、统一大脑（brain-core）和执行升级链讲成一套统一技术架构
aliases: [C2AI2X, ZhenIns, zhen-platform-core, zhen-brain-core, 刘智勇, 真机智能, 统一运行时, 多站执行主链, Agentic AI Summit]
tags: [ai-agent, architecture, practice]
sources: ["2026/08/11/前沿探索与超级Agent论坛/02-刘智勇-C2AI2X 协议与真机生态的统一技术架构.pdf"]
created: 2026-08-11
updated: 2026-08-11
---

# 真机智能统一运行时主链（C2AI2X 生态架构评审）

> 2026 Agent 大会（Agentic AI Summit 超级智能体系统架构峰会，2026-07-24/25，深圳）演讲。演讲者**刘智勇**（北京真机智能科技有限公司｜董事长兼首席科学家）。
>
> 本场目标：从 C2AI2X 协议切入，把真机生态里前台、平台、统一大脑和执行升级链讲成一套统一技术架构。本场为 100% 工程师受众、1 小时 CODE-FIRST 工程评审，评审对象是**一条多层执行主链**：垂直前台 → 平台网关 → 统一大脑。

## 开场判断与五个必须钉住的问题

> 问答已经越来越便宜，真正昂贵的，是问答之后如何继续执行。

5 个必须先钉住的问题（若不定住，后面的主链设计就会一直漂）：

| # | 问题 | 提问 |
|---|------|------|
| 01 | 权限 | 谁解释权限 |
| 02 | 状态 | 谁维护状态 |
| 03 | 升级 | 谁决定升级 |
| 04 | 执行 | 谁持续执行 |
| 05 | 兜底 | 谁在失败后兜底 |

## 真实在线主链全图

**前台编排 → 平台解释 → 统一执行 → 执行升级**：

| 前台编排 | 平台解释 | 统一执行 | 执行升级 |
|---------|---------|---------|---------|
| 浏览器 / 移动端 → zhenins-web / zhenins-backend；垂直前台 sample + 业务状态编排 | zhen-platform-core；key / scope / quota / entitlement | zhen-brain-core；preflight / agent loop / workflow | C2AI2X / Handoff → 异四 / 执行终端 |

评审结论：**前三层今天就已经在线承担统一运行时，后两层是执行升级方向**。今天先审前三层主链。

## 三层职责（运行时职责链）

三层不是并列功能模块，而是明确分工的运行时职责链：

- **编排层**（www.zhenins.com）：垂直前台 sample、垂直业务编排——先编排
- **解释层**（zhen-platform-core）：经营真相层、授权解释层、统一执行入口——再执行
- **执行层**（zhen-brain-core）：统一智能判断层（route / risk / agent loop、tool / memory runtime）

职责裁决：前台负责编排，platform 负责解释，brain 负责统一执行。

### 边界在仓库级落盘

边界不是口头约定，而是仓库级文档和职责归属的硬约束（三个仓库各有 AGENTS.md）：

- 垂直前台仓（www.zhenins.com）：业务前端不直连 brain-core
- 平台内核仓（zhen-platform-core）：`auth` 负责 Authentication Truth；platform-core 拥有 Platform Identity / billing / entitlement / key lifecycle truth
- 统一执行仓（zhen-brain-core）：brain-core 只接平台准入后的请求

## ZhenIns 前端与后端

- **入口契约**：01 `Next.js App Router`（Framework）、02 Chat-first 入口（Interaction）、03 公开获客页 + 咨询页 + 风险说明 + 角色入口（Surface）；状态也已开始拆层
- **ZhenIns 后端不是 thin API**：`public_chat.py` 垂直编排入口，含 7 个部分——session init、conversation CRUD、upload、SSE、consent、lead、coordinator status。复杂度主要不在模型，而在状态编排。

## 主链本质是标准异步工作流

用户入口 → 业务后端（POST `/execute` 开始请求）→ 平台网关（平台层解释权限与边界：key / scope / quota / entitlement、sandbox / trace / tenant）→ 统一大脑（授权后的执行请求：stream / tool / done）；GET `/workflows/:id/stream` 平台统一 query / stream / cancel。

### Single Ingress Rule：public ingress 只能有一个

平台层统一鉴权（01）、统一审计（02）、统一限流（03）、统一重写 stream / query / cancel URL（04）。业务后端不能直连 brain-core，本质不是工程洁癖，而是要把鉴权、审计、限流与 URL 重写收回平台层。

### Platform Core 不是 BFF

平台负责把业务请求翻译成可执行请求：校验 key / scope → 授权解释器 → 生成 quota、生成 entitlement → 生成 sandbox / tenant / actor / trace。输入：key / scope / domain / text；输出：授权执行请求。**关键不是 prompt，而是谁解释权限和边界。**

## 第一个真实问题：两套会话运行时

Runtime A（www.zhenins.com 的 public_chat.py）与 Runtime B（zhen-platform-core 会话 / 工作流运行时）并存。真正的问题不是重复代码，而是**会话主事实层到底放在哪一层**。

## Brain Core：双层执行结构

- **Ingress 契约**：brain-core 只接受平台放行流量。01 Command：`/execute`、`/reviews/resume`；02 Workflow：`/workflows/{id}`、`/workflows/{id}/stream`、`/workflows/{id}/cancel`；03 Trust：`x-zhen-platform-secret`
- **Brain Core 不是模型 API**：是 preflight + async runtime 的双层结构——sync preflight（validate access、sandbox、route、review gate）+ async agent loop（QueryEngine、tool use、memory、SSE stream）
- **preflight 的价值**：不是多一层，而是把不可逆执行前移成 deterministic gate。Stage 1 Deterministic Gate：`validateInput()`、`resolveDomainBinding()`、`enforceExecutionAccess()`、`validateExecutionSandbox()`、`analyzeDomainRoute()`；Stage 2 Execution Decision（awaiting human review / blocked / routed）。先 deterministic gate，再进 agent loop

### agent loop 已具备 AI 运行时特征

核心节点 `agent-loop-runner.ts`；执行核心：preflight、dynamic import、QueryEngine、MCP / tool pool；交互平面：history 注入、image / file block、SSE event emit。不是普通后端处理器，而是运行时执行容器。

## alias 共享：收益与代价

zhenhire / zhenlegal / zhenrent / zhenmem / zhenmeta / zhenquant 等站点统一映射到 `internalDomain = zhenins`，共享执行内核——**这是 alias 共享，不是执行网络统一**。

| 短期为什么很强 | 长期为什么危险 |
|---------------|---------------|
| 高复用：同一套 internalDomain / kernel / policy 可被多个入口复用 | prompt 泄漏、输出身份泄漏（`zhenins` review） |
| 高交付速度：新站点更多是在接入和配置，而不是从零再造 runtime | kernel 变成超级别名内核、policy 误复用；alias 一旦沉积成默认内核，边界就会持续变脏 |

结论：alias 是高效率策略，但不能成为永久结构，长期要收进 plugin 或更明确的 domain contract。

## 三个真实断点

1. **两套会话运行时并存**（见上）——主事实层未定
2. **metadata 尚未进入正式契约**：前台请求中的附带字段 `handoff_context`、`system_instruction_appended` 在平台正式契约处丢失，最终 Brain 入参缺失 `metadata.*`。字段出现，不等于契约接通
3. **迁移态还没完全收口**：旧链路还在（`brain_core_client.py`、`extended_search_service.py`、direct `bun run ... zhen-brain-core/scripts/*`）；状态面仍偏轻（platform-core 默认 file-backed state、brain-core 异步工作流默认内存态），replay / query 仍依赖轻状态面

## 必须拍板的 5 个决议

1. **决议 1 · 主事实**：会话主事实最终落在哪一层
2. **决议 2 · 边界**：平台运行时是否要按边界拆分
3. **决议 3 · 契约**：handoff 上下文是否进入正式契约
4. **决议 4 · 持久化**：工作流如何持久化 / 回放 / 恢复
5. **决议 5 · 共享**：多站共享如何从别名走向插件化

## 优化顺序与 30 / 90 天路线图

这不是功能优先级，而是架构裁决顺序：

- **第一阶段：收边界**——下线直连 brain 路径、决定会话主事实层、补齐正式契约缺口
- **第二阶段：稳运行时**——打通跨三层 trace、统一 workflow / stream / handoff 事件、补持久化 / 回放 / 恢复
- **第三阶段：共享产品化**——共享从别名走向插件、运行时与行业配置解耦

30 / 90 天路线图：**Phase A（30 天）**目标先把边界和契约收清：画清三层时序图、标明状态主事实层、收掉直连 brain-core 路径、补齐契约漏传字段；**Phase B（90 天）**目标让统一主链稳定、可追踪、可审计：会话运行时收口、工作流持久化稳定、链路支持追踪 / 回放 / 审计、共享开始走插件化。

## 主链已成立的 4 个信号

- **信号 1 主链边界**：正式主链边界已经基本对齐
- **信号 2 平台解释**：平台经营真相与授权解释已经成形
- **信号 3 执行结构**：brain-core 的 preflight / async 双层执行结构已经成立
- **信号 4 多站复用**：多站前台已经开始共用同一平台内核和统一大脑

方向已经基本成立，下一步重点是收口、持久化和治理，而不是推翻重做。

> 最后一句：真正难的不是单站回答，而是多站在同一条主链上持续执行，而且可解释、可追踪、可恢复。垂直前台不是重点，统一主链才是重点；不是单仓能力强，而是平台边界与主事实要稳；真正的增量在多站共用经营内核和统一大脑。

## 代码证据（备用页）

代码页只回答三件事：谁编排、谁解释、谁执行。

- **zhenins（2 个关键文件）**：`backend/app/api/v1/public_chat.py`、`backend/app/services/ai_integration.py`——证明点：前台编排
- **platform-core（5 个关键文件）**：`src/api/server.ts`、`src/runtime/platform-core-runtime.ts`、`src/entitlements/service.ts`、`src/keys/service.ts`、`src/store/file-platform-state-store.ts`——证明点：平台解释
- **brain-core（8 个关键文件）**：`src/api/brain-server.ts`、`src/api/execute-service.ts`、`src/api/agent-loop-runner.ts`、`src/api/workflow-engine.ts`、`src/api/brain-execution-registry.ts`、`src/workflows/expertHandoff/adapters.ts`、`src/workflows/networkHandoff/adapters.ts`、`config/domain-registry.json`——证明点：统一执行

## 相关页面

- [[agent-architecture-patterns]] — Agent 架构模式（preflight + async runtime 双层结构与漏斗式启动等模式的关系）
- [[agent-hook-governance]] — Hook 护栏治理（平台层统一鉴权/审计/限流的横切治理对照）
- [[agent-multi-agent-collaboration]] — 多 Agent 协作（多站共用统一大脑的执行网络）
- [[enterprise-agi-framework]] — 企业级 AGI 框架（受治理智能资产：平台经营真相层即企业本体的工程实现）
- [[ai-governance]] — AI 治理（权限解释、沙箱、审计的治理维度）
- [[next-gen-agent-form]] — 下一代 Agent 形态探索（同为 2026 Agent 大会论坛演讲）
