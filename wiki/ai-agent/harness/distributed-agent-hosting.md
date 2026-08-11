---
title: 从单体自管到分布式托管：腾讯云 Agent Runtime
description: 腾讯云 Agent 基础设施演进——三种单体形态（本地部署/K8s Pod/安全沙箱）→ 存算分离（Session/记忆/环境沙箱）→ 函数式托管与全托管 ReAct → Agent Registry/A2A 多 Agent 协作，核心论点「工作流是序幕，分布式托管基础设施才是终局」
aliases: [腾讯云 Agent Runtime, 分布式托管, Distributed Managed, Agent 架构演进, 从单体自管到分布式托管, Monolithic Self-Managed]
tags: [ai-agent, architecture, concept]
sources: [2026/08/11/Harness Engineering Agent执行与控制工程论坛/02-陈凯烨-工作流是序幕：Agent 从「单体自管」到.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 从单体自管到分布式托管：腾讯云 Agent Runtime

> 来源：2026 Agent 大会「Harness Engineering Agent执行与控制工程论坛」演讲《工作流是序幕：Agent 从「单体自管」到「分布式托管」》（From Monolithic Self-Managed to Distributed Managed），讲者陈凯烨（腾讯云）。

## 核心论点

> 工作流是序幕，分布式托管基础设施才是终局。

- Agent 正从「概念验证」走向「大规模生产化部署」，这是决定性拐点；但业界视线还停留在工作流编排与提示词调优——真正的工程挑战是：成千上万 Agent 长驻企业网络、自主调用工具、处理核心资产时，需要一个怎样的基础设施底座。
- 演进路径：**单体自管 → 分布式托管**；核心动作：先修安全，再谈分布式；先剥状态，再谈托管。

## 起点：单体 Agent（Chapter 01）

### Agent 是特殊工作负载——三个维度

| 维度 | 表现 | 对比对象 |
|------|------|---------|
| 自主性（AUTONOMY） | 主动决定「下一步做什么」，而非被动响应请求 | vs. 传统请求-响应服务 |
| 行为不确定性（NON-DETERMINISM） | 大模型驱动，同样输入也可能走出完全不同的执行路径 | vs. 幂等函数式服务 |
| 工具环境有依赖（ENV-COUPLED） | 通过工具调用感知与改造世界，环境即状态，环境即 Agent | vs. 无状态微服务 |

三大特殊性决定了 Agent 需要一整套全新的基础设施，而不是把普通服务框架改一改就够。

### 形态 A：本地单体部署（个人 Agent 起点）

场景：OpenClaw 本地部署 + 企微/飞书 IM 接入。四大痛点：

1. **可用性**：机器休眠/关机后无法使用，Agent 与个人电脑生命周期强耦合——合上电脑，服务就消失
2. **环境**：环境被破坏难以恢复，依赖、临时文件、密钥无处备份，出错要花几小时重装
3. **安全**：API Key、企业数据散落在个人机器上，审计和最小权限无从谈起
4. **管理**：企业看不到员工装了几个 Agent、在做什么，治理和合规是黑盒

### 形态 B：K8s Pod 化（企业 Agent 平台）

典型架构：一个 Agent = 一个 Pod + 挂 PVC 存状态；**10,000+ 同时长驻的 Agent Pod**。五大问题：

1. **隔离性不足**：容器级隔离可被突破，Agent 可能渗透到 Node，威胁其他租户与集群基础设施
2. **资源成本高**：Agent 是 IO 密集型，大部分时间处于空闲，Pod 常驻占资源
3. **可用性差**：单点应用，环境异常或 PVC 数据损坏后恢复困难
4. **难以运维**：升级期不可用、临时数据丢失、客户装的环境组件升级后突然失效
5. **性能有瓶颈**：当 Agent 从「一人用」变成「作为服务用」，单 Pod 撑不住并发压力

结论：Pod 化已经推进一大步，但把 Agent 当无差别工作负载塞进 Pod，无法真正解决架构问题。

### 形态 C：Runtime Sandbox 沙箱化

- **VM 级隔离，毫秒级启动**：冷启动延迟 80ms，按需拉起、用完即释放；Pod 冷启动通常在秒级，沙箱级别快了整整两个数量级。完美适配任务型 Agent（Coding、批处理）
- **秒级暂停/恢复**：暂停期不占资源（PAUSED · 0 CPU），恢复后内存、临时数据全保留；Agent 是 IO 密集型，大部分时间在等用户或等模型——成本从常驻到按秒

### Agent Wall：Sandbox 之外的安全总关

Sandbox 流量**透明转发**到 Agent Wall，三层网络防护 + 凭证注入统一在这里落地：

- **A 三层网络防护（3-LAYER NETWORK GUARD）**
  - LAYER 1：网络触达边界，只允许白名单出口
  - LAYER 2：站点防护策略，域名/路径/参数级细化
  - LAYER 3：敏感操作人工审核（Human-in-the-Loop）
- **B 凭证注入（ZERO-TRUST CREDENTIAL INJECTION）**：每个 Agent 实例都有可撤销的短期身份；出流量到 Wall 时按目标服务临时换发凭证；Agent 代码零硬编码密钥。最小权限、可撤销、可审计，符合零信任原则

### 单体还没到终点

安全防护住了，但三个宿命问题仍在：**运维难**（升级即中断，滚动更新有"用户会感知到"的窗口期，SPOF Ops Debt）、**高可用差**（有状态单点，状态在本地切主也切不走，Single Replica）、**无法横向扩缩容**（单沙箱扩不起来，No Horizontal）。结论：单体架构无解，必须走向分布式架构。

## 分布式 Agent（Chapter 02）

### 存算分离：分布式架构的核心

Agent 是有状态服务，存算分离后 Agent Loop 才能被当作普通无状态服务去调度、扩缩容、灰度、迁移。四个组件：

| 组件 | 定位 |
|------|------|
| Session Store（EVENT LOG） | 远端事件序列：高可用、TTL、回滚、分叉 |
| Memory Store（LONG-TERM MEMORY） | 从 Session 抽取的公共记忆，跨 Session 共享 |
| Environment Sandbox（TOOL EXECUTION） | 工具执行环境：快照回滚、秒级暂停恢复 |
| Agent Loop（STATELESS） | 无状态、可横向扩展；经 SDK/RPC 远程状态调用，按 SESSION_ID 按需拉取上下文 |

### 状态一：Session 剥离

- Session 是一个事件串，记录 Agent 与大模型/工具/用户的每一次交互；每次对话 Agent 把过往事件打包为上下文传给大模型——这就是"记忆"的物理形态
- 开源做法是本地文件 `.session.json`；分布式做法是远端存储，按 SESSION_ID 拉取
- **Cloud Session 五项增强能力**：
  1. **事件序列存储**：按顺序追加只增不改，强一致读写与快速查询
  2. **高可用·自动备份**：多副本存储 + 快照，数据丢失概率降到基础设施级别
  3. **TTL 自动过期**：按业务生命周期设置过期策略，成本合规都可控
  4. **Session 回滚**：回滚到任意历史事件点，Agent 犯错后可"reset 到刚才那步"
  5. **Session 分叉**：从某点复制出并行探索分支，支持"如果我当时…"式假设推演

- **Session 上云 = 数据平台**：事件流沉淀到远端后变成新数据源——① 公共记忆抽取（后台引擎抽取"这个用户是谁、爱好什么、约束是什么"写入 Memory Store，Async Job）；② 审计追踪（谁在什么时候让 Agent 调用了哪个工具、返回了什么，全部可回溯，Compliance）；③ 事件触发器（"Agent 一旦调用了 X 工具，就通知运维值班"，Sub/Push）。从"Agent 私有的记忆"变成"平台级数据资产"——这是存算分离带来的第一个红利

### 状态二：执行环境沙箱

> 工具调用在哪个沙箱，Agent 本质上就运行在哪里。环境沙箱是 Agent 状态最厚重的一部分。

1. **暂停恢复**：大部分对话不需要工具执行，沙箱暂停不占计算资源，恢复后内存、临时数据都在
2. **快照回滚**：任意点全盘快照，装错包/改坏配置秒级回滚到干净状态
3. **绑定语义**：与 Session / User 绑定，同一 User 跨 Session 对话感知同一环境——环境是记忆的物理容器

### 无状态 Agent Loop：K8s 部署后还不够

状态都抽出去后，Loop 直接当 Deployment 用 K8s 部署、前面挂 LB：

- **已解决**：水平扩展（想加多少副本加多少，突发流量性能瓶颈消失）、单点故障隔离（副本挂了会话可切到其他副本）、升级不丢数据（状态在远端，Loop 副本随便滚动升级）
- **待解决**：
  - Session 排他性：同一 Session 必须由单一 Loop 独占 → 需要 Session 抢主、分布式锁语义
  - K8s 集群运维负担：用户仍要管理 Deployment/HPA/Ingress/监控告警 → 每个 Agent 团队都要一位 SRE
  - 开源框架需改造：主流开源 Agent 都是本地状态假设，不能直接跑 → Runtime 提供云上 Session、远程工具 SDK 兼容

下一步：用函数式、全托管把剩余的运维负担推给平台。

### 函数式托管（Function-as-Agent）：用户只写 loop() 主函数

Agent Engine 核心架构：**Agent 管理与部署**（版本管理/配置管理/权限管理/监控观测）、**请求路由与编排层**（负载均衡/会话路由/策略路由/配额与限流）、**Agent Loop 实例池**（无状态服务：规划推理/工具选择/记忆读取/结果生成，自动扩缩容基于 QPS/并发/延迟等指标）、**Agent Memory**（集中化记忆层：Session 存储、长期记忆/向量库、配置/偏好/知识、记忆检索 API）、**Environment Sandbox**（远程执行环境，sandbox 按需唤醒/闲置休眠/状态保存/快速恢复）、**Agent Wall**（安全管控层：访问控制/流量审计/数据防泄漏/网络控制/安全策略/凭证注入/ACL 与动态凭证管理）、外部资源（互联网/SaaS、代码仓库 GitHub/GitLab、软件源 PyPi/npm/apt、对象存储/数据库）。流程：用户部署 Agent → 用户发送请求 → Agent Loop 调用工具（远程 Sandbox）→ 通过 Agent Memory 读写会话与记忆 → Sandbox 出口流量经过 Agent Wall → 结果返回用户。

用户代码（agent_loop.py）只包含：

```python
from agent_runtime import agent

@agent.loop(name="code_reviewer")
def run(ctx):
    # 只写 loop 主逻辑
    while not ctx.done:
        msg = ctx.llm(ctx.history)
        if msg.tool_call:
            result = ctx.tool.call(msg.tool_call)
            ctx.append(result)
        else:
            return msg.text
```

平台框架代管的 5 件事：**Session 生命周期**（SESSION_ID 路由、抢主锁、事件持久化）、**环境沙箱调度**（ctx.tool.call() 自动路由到用户绑定的 Sandbox）、**扩缩容 & 冷启动**（按请求粒度弹性、空闲缩到 0、秒级唤醒）、**可观测 & 灰度**（日志/追踪/指标/版本灰度开箱即用）、**部署编排**（git push 触发，无 K8s/无 Dockerfile/无 YAML）。

> 从写整个应用 → 只写一个函数。Agent Loop 的开发运维模型，本质上被"Serverless 函数化"了。

### 全托管 ReAct：用户不再写 Loop

大部分场景 ReAct 通用循环够用——用户只声明 System Prompt / Skills / MCP，Loop 由平台内置（agent.yaml，声明式，一键发布）：

```yaml
apiVersion: agent/v1
kind: Agent
metadata:
  name: code-reviewer
spec:
  systemPrompt: |-
    你是一个代码审查专家，
    找出潜在缺陷并给出建议。
  skills:
    - code_review
    - git_read
  mcp:
    - gitlab-mcp://internal
```

- 内置 ReAct 循环：REASON 思考 → ACT 调用工具 → OBSERVE 观察结果（LOCKED，用户不写）
- Prompt 直接映射到 Reason 阶段的角色约束；Skills / MCP 声明自动填充 Act 阶段可用工具集；循环控制、上下文压缩、错误重试全部平台代管
- Agent 从"应用"回到"配置"

### 托管度演进：四阶阶梯

| 阶梯 | 形态 | 用户交付 | 业界示例 |
|------|------|---------|---------|
| 1 | 单体式 Harness（Harness 随 Agent 应用运行） | 整个应用 | Claude Code / OpenClaw |
| 2 | 分布式 Harness（用户自管，部署在 K8s 等环境） | 用户服务（K8s/后端） | 元宝 / Manus |
| 3 | 函数式 Harness（云平台 Runtime 按需实例） | Brain Function / Handler | AWS AgentCore Runtime |
| 4 | 托管式 Harness（平台托管完整 Harness） | Agent 定义/配置 | Anthropic Claude Managed Agents |

越往上走用户负担越轻、标准化程度越高。

## 多 Agent 协作（Chapter 03）

单个 Agent 分布式化只是起点，Agent 之间还需要协作。三种基础能力对应三个物理层：

1. **Agent 互相发现（DISCOVERY）**：解决"我要用一个能写代码的 Agent，他在哪？"——类似传统微服务服务发现，但索引维度更丰富
2. **Agent 互相调用（A2A INVOCATION）**：解决"Agent A 怎样把任务交给 Agent B？"——Agent 也是 Tool，调用语义与 MCP 统一，复用同一套 SDK
3. **跨 Agent 上下文共享（CONTEXT SHARING）**：解决"多个 Agent 如何在同一件事上不重复对齐？"——STILL EVOLVING，目前多复用 Git Issue 或已有的 Human 协作平台

**Agent Registry**（可发现·可调用·可扩展的中枢）：① 注册中心（Agent · MCP · HTTP 服务统一注册）② 服务发现（Agent 也是 Tool，按能力检索）③ A2A 调用（通过 Registry 发起工具调用）④ 协议统一（MCP / HTTP / Agent → 统一 Tool）。**留白**：目前尚未提供上下文共享基础设施——用户仍复用 Git Issue 或其他 Human-to-Human 协作平台，是下一代基础设施的空白。

## 产品全景：腾讯云 Agent Runtime

- **典型客群**：大模型厂商（Agent 强化学习训练、评测）、Agent 服务商（Agent 构建与托管）、千行百业企业（AI+ 内部提效）
- **Agent 场景**：助手型 Agent（常驻·富状态）、领域 Agent（按需·有状态）、任务型 Agent（触发式·轻状态）
- **平台组件**：Agent Engine（部署与托管）、Agent Govern（治理与运维）、Agent Evals（评测）、Agent Sandbox（隔离执行环境：虚拟化/网络/存储隔离；多种类型环境：代码/手机/浏览器/桌面）、Agent Memory（记忆与智能，session）、Agent Identity（身份与凭证）、Agent Wall（入站管理：协议转换/沙箱路由/认证鉴权；出站管理：透明接管/策略管控/身份注入）、Agent Registry（Agents、Skills、mcp 注册、endpoints 管理）、Agent Gateway（模型与工具网关）
- **能力指标**：生命周期与状态管理（触发器、暂停恢复、会话级快照、分叉克隆）；弹性与效率（毫秒级启动、百万级并发、数十万镜像）

## 结论

1. **路径规范**：单体自管 → 分布式托管，是必经之路
2. **物理层规范**：存算分离 + 沙箱化，是最小公倍数
3. **协作层规范**：Registry + A2A，是多 Agent 时代的第一块基石

## 相关页面

- [[agent-harness-overview]] — Harness 综述：六承重层与七步循环
- [[harness-as-backend]] — Harness 作为新后端：Worker/Function/Trigger 三概念重译
- [[loop-engineering]] — Loop 工程：无状态 Loop 与反馈循环的对照
- [[harness-engineering-practice]] — Harness 工程实践
- [[harness-engineering-evolution]] — Harness 工程演进与企业落地（八大工程难题）
- [[agent-hook-governance]] — Agent Hook 治理：执行层的确定性约束
- [[openclaw]] — 形态 A 中本地单体部署的典型代表
