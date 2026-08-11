---
title: openJiuwen AgentOS——从 Model Scale up 走向 Agent Scale out
description: 金雪锋（2026 Agent 大会主论坛 Keynote）介绍 openJiuwen 通用 AgentOS：Agent 架构范式从 Context/Harness 演进到 Coordination/Symbiosis；多智能体原生（Swarm Skills/AutoGenetic Memory）、自演进原生（蜂群多参数演进）、分布式可靠运行时（百万级 Agent）、算力亲和（Agentic Inference KVC 管理）四大设计理念及社区生态数据
aliases: [JiuwenSwarm, openJiuwen, Agent Scale out, AgentOS, 蜂群智能体, Swarm]
tags: [ai-agent, tool, concept]
sources: [2026/08/11/主论坛Keynote演讲/03-金雪锋-JiuwenSwarm：从Model Scale up 走向 Agent Scale.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# openJiuwen AgentOS——从 Model Scale up 走向 Agent Scale out

> 演讲者：金雪锋（幻灯片未标注机构/头衔）；2026 Agent 大会主论坛 Keynote（DataFun × AGENTIC AI 超级智能体系统架构峰会）。内容为 **openJiuwen** 开源项目的介绍，三部分：Agent 技术的发展趋势 / openJiuwen 的设计理念 / openJiuwen 社区生态进展。

## Agent 技术的发展趋势：从个人效率工具走向企业生产级应用

架构范式随应用形态逐级演进：

| 应用形态 | Chatbot | 个人效率工具/数字助手 | 组织/团队级应用 | Mission-critical 应用 |
|---------|---------|---------------------|----------------|---------------------|
| **架构演进** | Context Engineering（上下文工程） | Harness/Loop Engineering（驾驭工程） | Coordination Engineering（协同工程） | Symbiosis Engineering（共生工程） |
| **核心问题/关键特征** | 模型无记忆、上下文长度易爆炸 | 单 Agent 长时任务完成率低、失败不恢复 | 多团队/业务流协同、企业级高确定性/高安全/分布式 | 高 Safety、低时延 |
| **模型演进** | LLM | Thinking / Agentic | 当前所处阶段 | 全模态 / World |

## openJiuwen 的设计理念

**定位**：打造**多智能体原生、自演进、企业级高可靠、算力亲和**的通用 AgentOS。生态覆盖行业智能体平台（金融、制造、政务、科研等 10+ 行业）、鸿蒙小艺/车/网络等、AgentArts 等。

### AgentOS 三层结构与设计理念

| 层级 | 组成 | 设计理念 |
|------|------|---------|
| **Agent Framework** | Harness、Coordination、Symbiosis、自演进 | **Skills as new Libraries** |
| **Agent Distributed Runtime** | Agent 发现互联、分布式状态管理、弹性多租 | **Agents as new Services** |
| **Agent System Service** | AgentOS 原子化系统服务（推理加速、Memory Storage） | **CLI/MCP as new Posix** |

关键能力标语：
- 首发 **Coordination、Symbiosis** 架构范式；Coordination Engine → Model Scale up → Agent Scale out
- **Swarm Evolving**：协同演进，越用越智能
- **分布式可靠运行时**：万级 Agent 协同，百万级并发
- **算力亲和**：KVC 主动协同，TTFT 15%↓
- **Jiuwen Symbiosis**：面向 Physical AI 的共生架构
- 底层硬件：昇腾、鲲鹏、超节点

### 多智能体原生：从 Model Scale Up 走向 Agent Scale Out

三方面支撑生产级 Agent 规模落地：

- **Swarm Skills（多智能体协作技能范式）**：① 原生的协作技能定义（角色/流程/约束）② 可控协同（Swarm Flow）③ 发现和共享（Swarm Skill Symphony/Hub）
  - 技能包结构：`SKILL.md`、`roles/`、`workflow.md`、`bind.md`、`dependencies.yaml`、`scripts/workflow.py`；分组为元数据（团队角色定义、协作流程）、边界约束（工具依赖）、可控执行脚本
- **Swarm 编排调度**：执行层 ReAct 与 Workflow 动静结合；任务层可抢占 Harness；协作层多 Agent 协同（自动+可控，SwarmFlow）、多模型路由
- **AutoGenetic Memory**：自主生长、分层沉淀的组织级记忆基因库；**LoCoMo 评测准确率较 OpenClaw 提升 15%，Token 消耗降低 60%**

### 自演进原生：全链路、多参数、蜂群协同

从**单 Agent 单参数演进**走向**蜂群多参数演进**，复杂任务成功率 **90%+**，智能体团队越用越好。

- 三条机制：① 执行结果、badcase 反馈学习，端到端调优 ② 提示词、上下文、记忆、工具多参数协同优化 ③ 文本梯度细粒度归因，精准定位优化元素
- 多参数、全链路演进：统一优化算子、精准回溯；多参数可扩展、异常可归因
- 团队组织与流程演进 → 基座 Harness（**组合寻优**）→ 专家 Harness（**能力蒸馏**）：基于协作流程优化群体协作流程和个体 Skills；基于任务场景寻找各成员 Harness 配置最优组合；垂域场景学习，蒸馏强大专家 Harness 配置

### 分布式可靠运行时：支撑百万级 Agent 高效可靠运行

| Agent 运行时挑战 | 弹性高可靠 Agent 分布式运行时 | 效果 |
|-----------------|----------------------------|------|
| **高动态**：高突发、长空闲、资源难预估/利用率低 | 分布式动态任务调度：分级调度、任务并行、自动弹性 | 大规模动态弹性：百万级 Agent 实例、千级 Swarm 实例、自动水平/垂直弹性、毫秒级唤醒 |
| **不安全**：高风险 AI 生成代码/工具调用 | 分布式多层次安全隔离：租户隔离、Host/Pod/沙箱隔离 | 精准安全隔离：风险任务动态精准隔离、多租户安全隔离 |
| **长会话**：长程有状态、故障后轨迹漂移/工具副作用 | 分布式状态管理和容错：Session 管理、语义一致故障恢复 | 长稳可靠：长程运行上下文一致故障自动恢复、断点续跑 |

运行时结构：**Agent Loop**（接收用户输入、LLM 大模型调用、Tool 工具调用——精准/安全/隔离、Code AI 生成代码执行）与 **Agent Swarm**（Agent → 子 Agent——动态/并发/调度）；基于高性能 **Session Event Log** 实现语义一致故障恢复。

### 算力亲和：KVC 主动协同、通算/智算统一调度

背景：Agent 长时运行、上下文频繁刷新，推理引擎堆积大量失效 KVC、缓存命中率低；不同 Agent 任务对模型、工具等资源需求不同，易导致某类资源高负载阻塞系统吞吐。

两个核心方案：
- **① Agentic Inference 主动式 KVC 管理**：HBM KVC 驱逐；多级 KVC 预加载/驱逐/卸载；KVC hint；推理引擎 + KVC 索引；HBM/DDR/SSD 多级缓存
- **② Agentic 异构任务感知调度**：模型分级与任务路由；负载感知/预测、性能感知、调度决策；多任务调度；算力集群调度引擎（K8S 等）；CPU/NPU 异构算力集群

效果验证（场景：代码全量分析、结构化报告生成；硬件 **910B4x2**（单卡 32GB 显存），模型 **Qwen3-32B**）：
- 30 并发下首 Token 时延降低 **22%+**；Prefix Cache 命中率提升 **20%+**
- ① Prefix Cache 命中率 **10%↑**、TTFT **15%↓**、TTLT **10%↓**；② 整体吞吐 **10%↑**

### Jiuwen Symbiosis：从数字世界走向物理世界

面向 **Physical AI** 的共生架构，破解三大挑战：**Safety**（物理世界不容试错，一步错即事故）、**Generalization**（换场景即失效，需重训，泛化难）、**Latency**（传统模型兼顾推理与运动控制，分钟级时延无法满足物理 AI 毫秒级需求）。

**Situation Awareness Loop（态势感知闭环）**：
`用户输入（自然语言、视觉、语音）→ Jiuwen Physical Perception 九问物理感知（目标检测、空间理解、结构化状态输出）→ Physical AI Agent（Observation & Feedback 观察&反馈：状态获取、视觉反馈、异常复位；Safe Planning 安全规划：意图理解&任务拆解、技能组装、安全校验）→ Physical Action 物理执行（原子动作执行：移动/抓取/放置；能力本地解耦：异常恢复）`

要点：感知闭环、运行时重规划；原子化工具库、组合泛化；规划执行分离、降时延；实时反馈与态势感知、异常自动回退；本体/环境/动作模块解耦；感知、规划、执行解耦；场景按需组合工具。

效果：松延动力（人形机器人）HDC 大会现场演示，配文"它现在进化能够解决长序列任务"。

## openJiuwen 生态与社区进展

### 智能体生态（携手行业伙伴）

- **通用资产**：**5000+ MCP**、**26万+ Skills**、MCP 权威市场、开源 Skills 中心
- **行业资产**：预置行业高价值资产——风控、理赔、深度研究 Agent；品控、售后 Agent；覆盖金融、制造等行业
- **四件套**：Agent 框架（Harness、Coordination Engineering）、Agent 运行时（互联协议、分布式运行时）、Agent 系统服务（沙箱、记忆存储）、典型应用（DeepSearch、JiuwenSwarm）
- 特点：通用/行业资产、极简开发、一键部署；**10+ 行业覆盖、50+ 客户商业部署**

### 生产级部署案例

1. **金融行业（邮储银行）**：与 openJiuwen 开源社区合作推出安全增强、自主创新的金融 **PSBC-Claw** 生态体系，已应用于情报监测、风险预警、技术洞察等场景，将逐步推广至办公、运营、信贷、风控等领域
2. **科研和学术（中科大"灵境造物"）**：MindSpore + openJiuwen，科学研究走向工程化、平台化和开放共享；模型库 SOTA 模型业界最佳适配、技能库 **300+ 多领域科研 Skills**、智能体科研全流程工作流编排
3. **华为云 OfficeAce**：企业级 Claw 应用，助力办公效率跃升；产品标签含 OpenClaw、OfficeAce、JiuwenClaw；依托 Harness 工程能力打造企业级智能体产品矩阵
4. **鸿蒙小艺开放平台**：繁荣鸿蒙智能体生态，全新升级

### 社区生态进展

- **社区活力**：Stars **36K+**、下载量 **170万+**、核心开发者 **550位+**
- **生态传播**：技术活动 **50场+**（顶会/双百工程/技术直播/交流会）、媒体曝光 **3亿+次**
- **行业应用**：金融、科研、制造、政务等；行业伙伴 **40+ 家商用落地**
- **已加入 Linux AAIF 成为金牌会员**，协同国际顶流社区发展
- 官网 https://www.openjiuwen.com/ ；开源代码仓 AtomGit（atomgit.com/openJiuwen）与 GitHub（github.com/openJiuwen-ai）

## 相关页面

- [[agent-multi-agent-collaboration]] — 多 Agent 协作模式（Swarm 协作与编排调度的理论基础）
- [[agent-design-paradigms]] — ReAct 与 Workflow 设计范式（Swarm 执行层"动静结合"）
- [[agent-memory-system]] — 记忆系统（AutoGenetic Memory 组织级记忆基因库）
- [[agent-mcp-protocol]] — MCP 协议（CLI/MCP as new Posix；5000+ MCP 资产）
- [[agent-harness-overview]] — Agent Harness 编排体系（架构范式演进：从 Harness 到 Coordination/Symbiosis）
- [[agent-architecture-patterns]] — Agent 架构模式（Harness/Loop 工程）
- [[claude-code-swarm]] — Claude Code Swarm 对等协作（Swarm 的另一工程实现）
- [[enterprise-agi-framework]] — 企业级 AGI 框架（Mission-critical 应用与受治理智能资产）
