---
title: AI 时代的数据基础设施展望
description: 腾讯云在 DataFun Agentic AI Summit 的展望——AI 时代多模态数据成为 Agent 的「认知燃料」，数据湖升级为 AI Native 智能数据湖计算（AI DLC），从「存好、算好结构化数据」转向「持续为智能系统提供可信上下文」
aliases: [AI时代的数据基础设施展望, AI Native 数据平台, Agentic Lake, 认知燃料]
tags: [big-data, ai-agent, concept, architecture]
sources: [2026/08/11/AI Native数据湖让企业Agent拥有统一的智能数据底座（腾讯云专场）/1. AI 时代的数据基础设施展望.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# AI 时代的数据基础设施展望

> DataFun Agentic AI Summit「AI Native数据湖让企业Agent拥有统一的智能数据底座（腾讯云专场）」开场演讲（全套幻灯片无演讲者署名，仅腾讯云品牌标识）。从两个案例出发论证数据平台的角色转变，提出 AI Native 数据平台架构与 AI 数据湖三大升级方向，最后预告腾讯云智能数据湖计算 AI DLC 全新发布（详见 [[tencent-ai-dlc]]）。

## 核心论点：数据从「运营记录」变为 Agent 的「认知燃料」

### 案例一：智能汽车

智能汽车从一次交付的机械工具，变为持续学习的智能系统。核心变化：**数据不断产生、软件持续迭代、AI 参与理解和决策**，汽车从一个产品变成一个持续运营的智能系统。

- 传统汽车：一台完成设计、制造、交付的机械产品，基本缺乏数据采集
- 智能汽车：一套持续感知、学习、升级并连接外部系统的软件定义平台

### 案例二：导购 APP

导购 APP 从一次性购买入口，进化为持续感知与主动服务的智能应用。当导购 APP 开始会理解用户，**数据不再只是运营记录，而是 Agent 获取上下文、形成判断与持续学习的认知燃料**。

- 传统导购 APP：一次性开发的标准化功能入口；用户使用以搜索/点击为主，运营以人工为主
- AI 原生导购 APP：持续感知、主动完成任务的智能服务体；语音/自然语言导购（语音导购、拍照推荐、智能运营、智能售后、SKU 运营），运营为 Agentic 运营
- 数据底座从单一结构化（订单交易数据、用户行为数据）扩展为结构化 + 多模态（Agent 记忆/轨迹数据、图像视频数据、商品百科数据、库存位置数据）并存的体系

## 数据平台的新使命（New Mission of Data-Platform）

AI 时代，**多模态数据将转变为 Agent 时代的「认知燃料」**——保证每一轮智能决策，都拿到**可追溯、可解释、可即时调用**的上下文。

完整链路：现实世界（人、设备、交易、环境）→ 持续数据流（实时采集、多模态融合）→ AI 理解与决策（LLM 上下文、Agent）→ AI 劳动力（Agent 替代人自主参与工作）→ 模型训练、持续优化。整个链路追求「越快、越准、越可信」。

## 下一代 AI Native 关键能力与架构

传统数据平台 vs AI Native 数据平台：

| 层 | 传统数据平台 | AI Native 数据平台 |
|----|------------|------------------|
| 应用 | 以数据消费为中心，向人提供查询结果 | **Agent 应用**：构建高可信、可持续优化的 Agent 应用，自主完成复杂工作 |
| 计算 | 单一预定义的精确计算 | **多模态计算**：低延迟完成 Agent 查询、检索、推理、代码执行等多模计算 |
| 存储 | 企业事实数据底座 | **Agentic Lake**：为 Agent 提供可信 Context、支撑持续推理并驱动行动 |

**AI Native 多模态计算引擎**覆盖：检索计算、精确 SQL 计算、模型训练计算、模型推理计算、上下文构建计算、测评和反思计算。

**Agentic Lake** 的核心是**统一 Context 上下文平面（Source of Truth）**，包含四层能力：
1. 多模态数据知识化存储
2. Agent 多级记忆管理
3. 上下文检索与组装
4. 反馈沉淀与持续学习

## AI Native 智能数据湖计算升级展望

**数据湖是 AI Agent 持续获得可信上下文、完成智能决策并自我进化的理想底座。**

AI 数据湖升级本质：从「存好、算好结构化数据」转向「**持续为智能系统提供可信上下文**」。

HOW：通过统一多模态数据与 AI 资产、融合计算范式升级，打造 AI 原生流水线，升级为持续进化的 AI 数据湖。三大升级：

| 升级方向 | 内容 | 价值 |
|---------|------|------|
| 可信上下文底座 | 数据、特征、模型、服务、Agent 轨迹/记忆的统一上下文 | 缩短从数据到智能的价值兑现周期 |
| 计算范式升级 | 建立 **Spark + Ray** 的统一资源、调度与运行时 | 减少系统拼接和上下文漂移 |
| Agentic 能力升级 | 让 Agent 彻底成为数据湖的新用户和新运维者 | 让昂贵算力持续做有效工作 |

## AI DLC 新架构（预告）

**腾讯云智能数据湖计算 AI DLC 全新发布**（详见 [[tencent-ai-dlc]]）：

- **Agent 应用层**：WorkBuddy、内容创作 Agent、智能营销 Agent、电商运营 Agent、法律顾问 Agent、智能驾驶 Agent、游戏运营 Agent
- **接入层**：Jupyter | VSCode | Web Shell | CLI | SDK | REST API
- **TCDataAgent-EVA**：腾讯云数据分析智能体
- **计算引擎**：Meson（Spark 生态的高性能计算引擎：Pipeline 执行模型、向量化算子）、Xpark（自研多模态数据计算引擎：多模态算子、ML/LLM Function）、TCRay（开源 Ray 全栈能力 + 全面内核增强：Ray Data 向量化/JIT 优化与读数优化、Ray Train Checkpoint 持久化与容错弹性优化、Ray Serve 国产 CPU 优化与自动伸缩、Ray Core 对象存储优化/集群容错增强/分布式调度增强）
- **TCinsight**：大数据智能管家（计算智能调优、存储智能调优）
- **统一多模态数据目录 TCCatalog**
- **统一存储**：ICEBERG（TCIceberg，湖仓数据）+ Lance（多模态 Lakehouse 存储）
- 应用场景：数据科学、智能搜索

## 相关页面

- [[tencent-ai-dlc]] — 本演讲预告的产品（AI DLC 四大升级与 Spark+Ray 一体化）
- [[tencent-ai-dlc-engines]] — AI DLC 三大核心引擎（TCRay/Xpark/Meson）深度解读
- [[lakehouse]] — 湖仓一体架构（Agentic Lake 的演进基础）
- [[agentic-data-cloud]] — Google Agentic Data Cloud（同为 AI 原生数据架构，System of Action）
- [[workbuddy-data-practice]] — WorkBuddy 基于 AI DLC 的数据实践
- [[workbuddy-context-engineering]] — WorkBuddy 上下文工程（AI DLC 新架构中 Agent 应用的上下文组织）
