---
title: 面向多 Agent 的容器化沙箱基础设施
description: 阿里云多 Agent 容器化 Sandbox 实践——安全隔离/极致弹性/状态保持三大挑战，OpenKruise Agents 生命周期管理（休眠唤醒/池化/Commit/Checkpoint），MiniMax 与 Kimi 客户案例
aliases: [Agent Sandbox, 容器化沙箱, OpenKruise Agents, Agent-Sandbox, 袁瑞杰]
tags: [ai-agent, practice, tool]
sources: [2026/08/11/多Agent协作论坛/02-袁瑞杰-面向多 Agent 的容器化沙箱基础设施实践.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 面向多 Agent 的容器化沙箱基础设施

2026 Agent 大会「多Agent协作论坛」演讲（演讲嘉宾：袁瑞杰），主题为基于容器的 Agent Sandbox 基础设施实践，覆盖五个部分：基于容器构建的 Agent Sandbox、关键技术实现、阿里云开源的 OpenKruise Agents、生态与客户案例。

## 背景：智能体方案简图与业务挑战

阿里云容器服务智能体方案简图：用户 / Agent（Agent 2 Agent）→ AI Agent（ReACT/CoACT…）→ 大模型（Planning/Inference）；记忆（短期/长期记忆）→ 记忆库、知识（向量库/检索）→ 知识库、工具；MCP Client/MCP Protocol → MCP Servers 让工具使用更高效；Sandbox（Code Interpreter / Browser Use / Compute Use）实现环境隔离、安全执行、极致弹性；ACK Agent Extension 提供 Sandbox Resource，ACR 提供 Sandbox Templates（Code Interpreter / Browser / Compute Use）。术语：ACK = 阿里云容器服务 Kubernetes 版；ACS = 阿里云容器计算服务；ACR = 阿里云容器镜像服务。

智能体应用落地面临三大业务挑战（Sandbox：高安全性、状态保持场景）：

| 挑战 | 要点 |
|------|------|
| **数据安全** | 攻击者提示词诱导恶意行为，模型动态生成不可预期代码；多会话数据需要严格隔离 |
| **大规模极速交付** | 模型动态控制工具的执行，存在更大规模的秒级交付弹性并发；会话数量、并行度加剧资源需求动态波动 |
| **状态持久化** | AI Agent 长周期/多步骤/强状态，跨多轮交互与工具调用；多会话并存加剧沙箱规模膨胀；不是所有会话时刻存活 |

结论：算力需要运行在安全隔离的环境；算力要极致弹性能力；状态要能有效保持；成本要能合理控制。

## 沙箱生命周期

状态机：Create template → Pending → Running → Completed；Pause/Resume → Paused（Generate new template 可从 Checkpoint 生成新 Sandbox Instance）。

- 01 模板：包括镜像、编排和可选的 checkpoint
- 02 Paused：资源占用最小化（无 CPU 等消耗）
- 03 Checkpoint：内存、临时存储和显存状态数据

## 关键技术实现

### 安全隔离（应对数据泄露、代码注入、网络攻击等风险）

| 维度 | 实现 |
|------|------|
| 计算隔离 | CPU/内存相互隔离，互不干扰 |
| 网络隔离 | 禁用东西向网络；南北向单向连通；独立公网访问 |
| 存储隔离 | 共享存储的挂载点隔离 |
| 鉴权隔离 | 单 Agent 的独立 rbac 鉴权 |
| 可观测 | 所有 Agent 行为可追踪、可审计 |

### 大规模极致交付的容器资源管理

Sandbox 资源管理的复杂度：运行时间短、对启动速度要求高；需等待人类或其它工具反馈，等待时间长，整体生命周期时长难预测；资源消耗难预测；极致弹性的技术实现复杂度高。

- **Serverless 算力**：业界典型方案阿里云 ACS Pod、AWS Fargate、Azure AKS Pod、GKE Agent Sandbox——绝大部分 Agent 用户倾向优先使用 Serverless 以简化 Sandbox 使用复杂度
- **K8S 节点池**：业界典型方案阿里云 runD、Kata/kata on PVM、Firecracker、gVisor——安全沙箱技术、二次虚拟化技术（计算、存储、网络隔离）
- **算力层**：物理算力底座 AMD EPYC / Intel Xeon；高核密度单颗最高 192C/384T、高整合比约 7:1、TCO 最高降 67%，支撑海量沙箱高密部署与低成本弹性

### 状态保持技术

- 文件系统的数据保持：容器 rootfs 数据、临时卷、持久化卷的数据检查点保留能力
- 内存/显存数据保持：CRIU、VM Memory Checkpoint 等技术实现内存数据的快照点保留能力；NVIDIA/cuda-checkpoint 等显存保持能力
- 两种开源方案思路：firecracker（文件系统使用数据持久化存储 + VM 维度的内存 Checkpoint）；gvisor（文件系统使用数据持久化存储 + gvisor 的内存 CRIU 技术）

### ACS Sandbox 极速唤醒工程实践

1. 池化技术：同构沙箱配置预定义，自动匹配实现极速启动与唤醒；预热池管理产品化（指定 Pool id 自动匹配）
2. Serverless 模式：具备大规模极致交付能力的 Serverless 算力，简化 Sandbox 资源管理
3. 基于已有的 ACS 安全沙箱隔离技术、网络、存储隔离能力
4. 基于阿里云块存储的快照预热与复制能力，实现极速 Rootfs 恢复
5. Sandbox Operator 简化客户使用方式，管理 Sandbox 生命周期；屏蔽预热池管理复杂度

架构：ACK/ACS 集群 → Sandbox Operator → 沙箱配置预定义 → Pool-1/2/3（for 业务 A/B/C）；ACS 全托管预热池包含 MicroVM（计算规格、网卡、存储设备预热）、RootFS（ACS rootfs 缓存预热缓存预…）、容器镜像缓存（ACS 镜像缓存自动创建挂载）。Sandbox Operator 生产实践经验进一步开源到 OpenKruise Agents 社区。

### 休眠唤醒运行时生产经验（两种方案）

- **Solution 1：CRIU 容器环境休眠唤醒**（内存 Dump → 进程级内存数据 → Lazy Load 秒级恢复）：休眠轻量、数据量少、休眠快；内存状态管理数量少，仅需考虑业务程序的状态；硬件耦合度较低
- **Solution 2：虚拟机快照整机休眠唤醒**（整机内存数据 → Lazy Load 秒级恢复）：虚拟机休眠技术更成熟、稳定性好；硬件耦合度较高、不支持跨代际

## OpenKruise Agents（阿里云开源）

Agent-Sandbox 是 CNCF 孵化项目 OpenKruise 社区开源、面向 AI Agent Sandbox 的端到端解决方案，提供标准化的 Sandbox 生命周期管理能力。项目地址：https://github.com/openkruise/agents

### 定位与能力

- 管理 Sandbox 生命周期的标准能力：Sandbox 创建、休眠、唤醒（包括内存、读写层数据等）；高效资源供给（资源池化和资源变配，加速沙箱冷启动）；Checkpoint/Fork 能力，满足 RL 训练任务
- AI Agent 与 Kubernetes 的中间纽带：向上通过 E2B 等协议对接 AI Agent（E2B SDK/MCP → Agent Proxy → Agent-Apiserver）；向下通过 K8S 协议，使用 Pod 承载 Sandbox（Kata / MicroVM / Virtual Kubelet）

### AI Agent 集成方式（两种）

- **Python SDK（E2B 兼容）**——面向 AI 科学家/开发者：Agent-Apiserver 支持通过 E2B 等协议的 REST/gRPC 接口，无缝对接 E2B 原生 Python SDK；支持 `create()`、`sleep()`、`wake()`、`destroy()` 等高级生命周期操作；无需了解 Kubernetes 细节，以代码原生方式管理 Sandbox 实例
- **Sandbox CR**——面向 Agent 平台/运维工程师：根据 Sandbox CR 自动完成 Pod 创建、状态同步、休眠调度与资源回收；类似 Deployment，通过 CRD SandboxSet 声明期望的 Sandbox 实例数量与配置（`apiVersion: agents.x-k8s.io/v1alpha1`）

### 三大组件

| 组件 | 能力 |
|------|------|
| Agent Apiserver | 支持社区 E2B 等 API；Sandbox 路由管理 |
| Sandbox 管理 | Sandbox 休眠/唤醒；Sandbox 池化 |
| Checkpoint 管理 | Pod 的快照/克隆；容器的镜像 Commit |

### 休眠唤醒流程

1. 通过 E2B 休眠 Sandbox → 2. 通过 Job 触发 Sandbox Pod 休眠 → 3. 保存 Sandbox 状态到云盘（PV、Snapshot）→ 4. 调用模型推理 → 5. 唤醒 Sandbox Run Code → 6. 通过 Job 唤醒 Sandbox → 7. 通过云盘恢复 Sandbox 状态。

- 休眠（Sleep）：空闲时转入休眠状态，降低成本；唤醒（Wake）：再次需要执行任务时快速恢复 Sandbox 到休眠前的状态
- Sandbox CR 示例包含 `pause: false` 与 `persistentContents`（memory / filesystem / ip / gpuMemory）

### 池化扩容与降本

- 快速获取：预热池中秒级分配就绪实例，避免冷启动延迟；快速路由：内置 Agent-Proxy（Envoy、Proxy-Manager）边车对 Sandbox 路由；池内分 In-Using / Available / Paused 三类
- 降低预热成本：AutoScaler（基于预热池水位的弹性 + 基于时间的弹性）、Refresher（已使用的 Sandbox 复用 + Sandbox 镜像的动态更新）
- PoolingAutoScaler CR 示例：`observeWindowSeconds: 600`、`minReplicas: 3`、`maxReplicas: 100`、`minAvailableRatio: 20%`、`maxAvailableRatio: 90%`；基于时间调度可到 `minReplicas: 10`、`maxReplicas: 150`

### 状态保持：Commit 与 Checkpoint

- **Commit CR**（持久化文件系统读写层）：通过 CRI 接口捕获 Sandbox 的 overlayfs/rw-layer；将运行时文件系统差异层打包为标准 OCI 镜像；实现 Sandbox 状态的可回溯、可重建、可迁移。流程：Commit CR → 创建 Job Pod（挂载运行时 sock 文件）→ 通过运行时将文件 diff 打成 Tar 包 → 将 Diff 构建成镜像推到仓库 → 通过 Commit 镜像创建新 Sandbox → 从仓库拉取。CR 含 `image`、`ttl: 72h`、`registryAuth` secrets 等
- **Checkpoint CR**（持久化内存和文件系统读写层）：文件系统层调用 CRI 接口导出容器 rw-layer 为标准 OCI 镜像；内存与进程状态通过 CRIU 执行 checkpoint，保存寄存器、内存页、网络连接等上下文；输出一个可重建的完整 Sandbox 快照（镜像 + 内存 dump）；当前通过镜像 commit 实现，未来通过云盘实现。CR 含 `keepPodRunning: false`、`ttlAfterFinished`、`persistentContents`（目前仅支持 memory、filesystem 两种，默认启用 memory、filesystem）

## 生态集成

- 分层生态：Agent Framework（LangGraph、AgentScope、Kagent、Dapr Agent）；Sandbox Infra（OpenKruise Agents、AgentCube、SIG Agent-Sandbox）；Agent Runtime（AIO Sandbox、redroid、AndroidWorld）；Container & Virtualization（Kata、gvisor）
- **AgentScope on K8s**（https://github.com/agentscope-ai/agentscope）：构建、编排和运行 AI Agent/多智能体应用的开源框架与工具集；一键部署 Agent APP（基于 AgentScope Core 框架）到 Kubernetes 集群（ECS/Serverless Pod；Kata / MicroVM / Virtual Kubelet）
- 智能体应用 on K8s 整体概览：百炼生态（AgentScope/百炼插件）、Langchain、自建应用层；K8s 生态走 Pod 协议、E2B 等生态走 E2B 协议容器兼容层；底座为 Kubernetes API 与阿里云容器服务（ACK/ACS）

## 客户案例

### MiniMax：基于 ACK + ACS 重构 Agent 运行底座

MaxClaw / MaxHermes——控制平面与执行平面分离，海量 Agent 从「可用」走向企业级「落地」。

- 控制平面·阿里云 ACK：消息分发、任务编排、调度、策略下发、状态管理·运行观测
- 执行平面·ACS Agent Sandbox：MicroVM 沙箱；按需拉起·分配·回收；独占 ESSD / 弹性网卡 / Checkpoint

关键数据：**20~40ms 极速实例供给**（模板预热 + MicroVM 轻量虚拟化，远优于传统容器数十秒冷启动）；**15000 沙箱/分钟**（潮汐流量下大规模弹性供给，按需创建、用完即释放）；**MicroVM 级强安全隔离**（独立内核 + ESSD 加密盘 + NAS + 网络 Default Deny，风险收敛实例内）；**状态不丢长任务连续性**（分层持久化 + 运行时 Checkpoint，实例漂移/重启后快速重建上下文）。数据来源：阿里云 × MiniMax 公开技术合作披露。

### Kimi（月之暗面）：基于 ACK + ACS 的 Agent Infra

深度研究 / OK Computer 等 C 端 Agent + K2 模型 RL 训练——常态算力与 Serverless 分级调度，兼顾性能、成本与稳定。

- 业务场景 & 痛点：C 端高峰数万并发、每个请求秒级要独立算力、容器分钟级冷启动不可接受；执行大模型生成代码未经人工验证、多租户下必须硬隔离；研究型长任务易断线（中间数据/推理路径一旦销毁，用户被迫从头再来）；RL 训练批量启停、峰值预置资源浪费巨大
- ① ACK 节点池·常态基线：多可用区/多规格即时弹性、自定义镜像 + 数据盘快照、Terway ENI 预分配 → 千节点分钟级扩容
- ② ACS Agent Sandbox·Serverless 溢出：MicroVM 轻量虚拟化秒级拉起数千沙箱；Quota 热更新 burst；休眠唤醒；内存级 Checkpoint 克隆（排队 > 500 / 等待 > 30s → 溢出）

关键数据：**90% 虚拟化开销降低**（数千沙箱秒级并发启动）；**60%+ 沙箱启动提速**（镜像缓存 + Quota 热更新 burst，Python 类沙箱启动时间缩短过半）；**10万+ Pod 规模稳定调度**（调度器性能达开源数倍，千节点每秒数百 Pod 稳定运行）；**数秒克隆 RL 分支路径探索**（内存级 Checkpoint 快照，瞬时克隆数千副本供 MCTS 探索）。数据来源：阿里云 × Kimi（月之暗面）公开技术合作披露。

## 相关页面

- [[agent-multi-agent-collaboration]] — 多 Agent 协作模式（沙箱服务于多 Agent 执行）
- [[agent-harness-overview]] — Agent Harness 综述（Sandbox 属运行环境层）
- [[agent-mcp-protocol]] — MCP 协议（沙箱上层的工具接入）
- [[ai-agent-security]] — AI Agent 安全（沙箱安全隔离实践）
- [[agent-architecture-patterns]] — Agent 架构模式
