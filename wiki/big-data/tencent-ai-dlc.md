---
title: 腾讯云智能数据湖计算 AI DLC
description: 腾讯云数据湖计算 DLC 升级为 AI DLC——Spark + Ray 一体化、四大产品升级（计算范式 Ray x Spark、引擎 Xpark + Ray Libs x Meson、元数据 TCLake、Agentic MCP x Skills x CLI），从大数据计算平台进化为原生支撑 Agent 运行并自我进化的智能底座
aliases: [AI DLC, 智能数据湖计算, Tencent AI DLC, DLC 发布, Spark+Ray 一体化]
tags: [big-data, ai-agent, tool, concept]
sources: [2026/08/11/AI Native数据湖让企业Agent拥有统一的智能数据底座（腾讯云专场）/2. 腾讯云智能数据湖计算 AI DLC发布.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 腾讯云智能数据湖计算 AI DLC

> DataFun Agentic AI Summit「AI Native数据湖让企业Agent拥有统一的智能数据底座（腾讯云专场）」发布演讲，主题为「AI Native 数据湖的 Spark + Ray 一体化实践」。腾讯云数据湖计算 DLC（Data Lake Compute）升级为**智能数据湖计算 AI DLC**：从大数据计算平台，进化为原生支撑 Agent 运行并自我进化的智能底座。四大引擎/组件细节见 [[tencent-ai-dlc-engines]]，整体展望见 [[ai-native-data-platform-vision]]。

## 趋势与洞察

### 智能计算范式的演进（数据 · 计算 · 训练）

每个计算时代都有专属的计算框架与引擎；每次范式更迭，都在淘汰旧架构、兴起新计算：

| 维度 | PC 互联网（2000s, Web Services） | 移动互联网（2010s, 云计算+大数据） | AI/Agent 时代（2022s+, 智能涌现） |
|------|------|------|------|
| 数据形态 | 结构化交易数据 | 用户行为大数据 | 多模态数据 |
| 计算资源 | CPU 纵向扩展、一体机 | CPU 多核、横向扩展、分布式 | GPU + CPU 异构、分布式 |
| 计算引擎 | Client-Server、LAMP 技术栈 | K8s、Spark / 云数仓 | AI 原生计算平台 |
| 训练范式 | 监督学习、浅层模型 | 深度学习、模型训练 | 预训练 → 后训练定制 SFT/RLHF/RL |

三个核心洞察：
1. **数据处理本身成为异构工作负载**——多模态打破单一框架边界
2. **企业转向后训练**——预训练集中于头部厂商，企业用私有数据定制开源模型
3. **转变速度决定生死**——每次范式更迭淘汰旧架构、兴起新计算

### 产业观察：两个阵营走向彼此

过去是大数据平台里「加 AI 功能点」；这一次，是两个阵营带着各自的解决方案，认真地走向彼此：

- **大数据阵营**：Databricks（湖仓一体，收购 MosaicML，自研大模型与 Agent）、Snowflake（云数仓，Cortex AI 套件、Arctic 开源模型）——擅长数据治理、SQL 生态、企业级可信
- **AI 阵营**：Anyscale（Ray 商业化，Data + Train + Serve 一体；开源 Ray，AI 计算框架长出 Ray Data 大数据处理）——擅长 Python 原生、异构算力、框架亲和

判断：**大数据与 AI 各自擅长、天然互补——当两个阵营站到同一张桌前，谁能把「融合」做成产品，谁就定义下一代数据平台。**

### 数据工程方式的转变（开发 · 分析 · 服务）

确定性系统「一次写对」（逻辑一次写进代码，测试通过即永远正确）vs AI 系统（输出概率化、数据会漂移，不可能一次写对；**迭代速度 = 感知-判断-修复的端到端延迟 = 竞争力本身**）：

| 环节 | 传统 | AI |
|------|------|-----|
| 开发 Development | SQL/ETL 声明式，关系代数计算，产出结构化的表 | Python-first + 异构计算 pipeline，产出可训练语料和 Agent 轨迹、记忆与上下文 |
| 分析 Analysis | BI 报表/OLAP，回答「发生了什么」 | AI 特征工程 + 后训练定制，让模型学会分析并驱动 Agent 决策 |
| 服务 Service | 数据服务 API 取数，一次上线即稳定 | AI 模型即服务 MaaS，Agent 运行时闭环进化 |

开发、分析、服务正在合成一条 **Agent 持续进化的链路**——数据、模型、算力每个维度都在快速迭代，而这条链路的运转速度决定了 AI 的业务效果；数据平台的演进方向，就是让这条链路转得更快。

## 新平台的四条设计思路

AI 时代，必然需要属于自己的计算框架与计算引擎：

1. **支持异构（Heterogeneous-Native）**——硬件从 CPU 到 CPU + GPU，数据从结构化到多模态，集成异构计算框架
2. **AI 原生开发（AI-Native by Design）**——以 Python-first 与分布式框架为原生底座，保持开放生态，让开发、分析、服务回归 AI 工程师的习惯与效率
3. **数据算力同底座（One Unified Foundation）**——训练、分析、推理共享同一份数据，闭环就地自转
4. **面向 Agent 进化（Built for Agent Evolution）**——Agent 轨迹与反馈就地沉淀回流，平台服务于「链路运转效率」，而非一次性的任务执行

## AI DLC 四大产品升级

DLC → AI DLC 的四项升级（计算范式、引擎性能、元数据、Agentic 能力）：

| # | 升级 | 内容 |
|---|------|------|
| 01 | **计算范式升级（Ray x Spark）** | 一份算力同时驱动数据处理、大模型后训练与在线推理；开放兼容主流开源引擎，客户技术栈平滑迁入 |
| 02 | **引擎能力升级（Xpark + Ray Libs x Meson）** | Xpark 向量化引擎加速 Ray Data，配合 Ray Train/Serve 覆盖数据处理、训练、推理全链路 |
| 03 | **数据算力同底座（TCLake）** | 从数据 Catalog 升级为数据对象与 AI 对象的元数据与存储底座 |
| 04 | **Agentic 能力升级（MCP x Skills x CLI）** | 自然语言驱动智能分析与系统诊断，底座本身可被 Agent 编排调用 |

### 支持异构（Heterogeneous-Native）

AI 时代的资源天生异构。平台在计算与存储两端，把异构算力与多模态数据统一纳管、按需调度：

- **计算资源（异构算力池）**：① 多元异构算力——CPU/GPU 统一纳入同一资源池，屏蔽硬件差异；② 细粒度动态调度——同一份算力同时跑数据处理、后训练与在线推理，按需分配；③ 高速互联加速——RDMA 等加速设施，提速 GPU 间上下文交换与集合通信
- **存储资源（异构高性能存储）**，按 AI 全生命周期组织：① 训练——高吞吐加载，海量多模态数据 + TB/s 带宽加载权重与 Checkpoint；② 推理——低时延状态，KVCache/Embedding 高速流转，毫秒级命中支撑高并发推理；③ Agent——就地沉淀回流，轨迹/记忆/产物就地留存，可检索、可回流为下一轮语料

资源层的使命：把异构的算力与异构的数据，变成一个可统一调度、按需伸缩的资源底座，让上层引擎无需关心硬件与格式差异。

### 双引擎：DLC 数据计算 + DLC AI 计算

资源层统一容器化资源调度（异构算力池、统一调度、自动扩缩、滚动升级、故障自愈），引擎任务由调度层按需拉起 Pod，按时长/资源计费。

| | DLC 数据计算（100% 兼容 Spark，自研内核全面升级） | DLC AI 计算（100% 兼容开源 Ray） |
|---|---|---|
| 定位 | 数据并行、结构化与半结构化处理 | 任务并行、异构算力、AI 原生 |
| 场景 | 数据加工与数仓建设（大规模表 join/聚合/清洗）、特征工程与数据准备、大规模机器学习（Spark MLlib）、BI 分析与报表 | 多模态数据处理（视频/图像/文本，CPU+GPU 异构管线）、大模型后训练与推理（SFT/RLHF/RL、在线推理服务）、Agent 运行时（多步推理、工具调用、长期记忆与协作）、强化学习与超参搜索（RL、复杂计算任务、HPC） |
| 性能 | 自研向量化引擎：1TB TPC-DS 性能 3.6X、CPU 占用下降 50% | 内核优化 + 平台增强：Ray API 零改动，性能大幅领先开源 |

双引擎如何配合：① **Serverless 化计费**——按任务/时长计费，免集群运维、免资源预留；② **数据不跨系统**——Spark 与 Ray 共享同一份资源池与数据底座；③ **统一调度与治理**——数据与 AI 计算任务统一管控、统一权限与配额；④ **统一可观测**——日志/指标/追踪打通，无需跨系统排查。

### 三大引擎发布概览

- **TCRay**：面向 AI Workload 的 CPU + GPU 统一调度底座——开源 Ray 的全栈能力 + 全托管的工程化体验（内核优化、平台能力、可观测一应俱全），详见 [[tencent-ai-dlc-engines]]
- **Xpark**：DLC 多模态计算引擎——基于 Ray 框架构建、SQL-First AI Function、50+ 多模态/LLM/ML 算子；推理吞吐相比开源 Data-Juicer 提升 3 倍，GPU 资源利用率长期稳定接近 100%
- **Meson**：DLC 数据计算向量化引擎——1TB TPC-DS 查询分析场景性能提升 3.6X，腾讯云增速最快的自研计算引擎，100% 兼容开源 Spark、原生支持增量计算

### TCLake：智能 Data + AI 统一元数据层

纳管数据对象与 AI 对象，从数据加工到模型服务全流程的治理与血缘。全流程血缘四环节：① 数据加工（表/文件/Topic、作业运行状态、数据质量）→ ② 特征工程（特征集+版本、特征血缘、特征统计）→ ③ 模型训练（模型+版本、训练作业+Checkpoint、超参数+指标）→ ④ 模型服务（服务端点、推理日志、Agent 轨迹/记忆）。

四大能力：
1. **统一纳管（Unified Catalog）**——数据对象（表/文件/Topic）+ AI 对象（模型/特征/Agent 轨迹）一套模型统一管理
2. **直接注册（Direct Registration）**——引擎主动注册元数据与运行状态，非被动采集；变更即时双向同步
3. **端到端治理（End-to-End Governance）**——访问控制、审计、发现、血缘追踪，从数据到模型全链路可追溯
4. **多引擎接入（Multi-Engine）**——Spark / Ray / Xpark / Meson 等数据与 AI 引擎统一接入，运行状态自动归集

### Open Engine：面向 Agent 进化

Open Engine 让客户 Agent 持续进化 + MCP/Skills/SDK 让平台被外部 Agent 调用 + TCinsight 智能自治。

客户 Agent 进化闭环：客户 Agent 接入（GPU 资源 + Agent Runtime：Ray Data / Ray Serve）→ 过程数据采集（Agent 运行产生的轨迹、记忆、反馈）→ 数据处理 + 特征提取（进入数据处理管道，提取可用特征）→ **Ray Train 后训练反哺**（回归反哺模型，让 Agent 越用越聪明）→ 持续进化。

- **MCP（模型上下文协议）**：平台被外部 Agent 调用——WorkBuddy、CodeBuddy、Claude Code 等
- **Skills**：技能原子化
- **SDK / Open API**：完整编程接入
- **TCinsight 大数据智能管家**（智能自治：自感知/自诊断/自处理/自恢复；含自主调优 Agent、自主运维 Agent、预测治理 Agent，底座库为模型库/场景库/策略库/数据特征库/多维多层库）：资源降低 15%、根因定位 4.5h → 30min、VLDB 2025 收录
- **全链路系统智能**：每个阶段都有 Agent 协助用户——购买选型（资源规划 Agent）、迁移上云（智能迁移 Agent）、日常使用（智能管控 Agent）、运维调优（智能自治 Agent）

## 产品演示（实际功能界面）

- **Open Engine 功能**：新建计算引擎支持三类——SQL Warehouse（Spark SQL 交互式分析）、Ray（训练/推理/分布式 Python）、Open Engine（MLflow 实验跟踪：实验元数据 + Artifact 存储 + 模型注册中心；Agent 服务托管：部署托管 Agent 应用，内置可观测与评测集成）
- **基于 Trace 数据发起后训练任务**（底层执行 Ray Train，训练完成自动入库）：任务列表含 sql-bot（DPO 偏好优化，基础模型 Qwen2.5-72B，数据集「近 30 天失败 + 低分 Trace（328 条）」）、rag-helper（SFT 监督微调，Qwen2.5-7B，「产品 RAG 问答标注集（1,240 条）」）、fin-helper（SFT 监督微调，Llama-3.1-70B，「财务领域监督微调集（3,200 条）」）、sql-bot（Qwen2.5-72B-sql-bot-dpo-v3，「在线交互奖励（点赞/点踩，2,800 条）」）——用 Trace 数据（失败/低分/反馈）直接发起 Agent 后训练
- **WorkBuddy 慢任务优化**（AI DLC 功能：使用 Agent 工具 WorkBuddy 操作 AI DLC）：同一条 SQL 冷引擎 vs 热引擎实证对比——总耗时 16.5s → 7.1s（-57%）、实际计算 8.6s → 1.7s（-81%）、排队 4.5s → 1.6s（-65%）；约 80% 的慢是引擎冷启动（executor 缩到 0 后重新拉起）造成的，数据量本身不是瓶颈；给出 5 条优化建议（保持引擎温热、别用 SuperSQL 3.5 引擎跑交互 SQL、高频 COUNT 做物化汇总表、表变大时再建明确 schema + 分区/排序、排查流式任务空转）
- **orders 数据看板**（WorkBuddy 导出重庆区域 orders 表 49,990 行并生成看板）：总交易额 GMV ¥1.44 亿、日均约 ¥39.3 万、客单价 ¥2,871.21、活跃用户 1,000、人均消费 ¥143,532、商品数 100、累计销量 149,478 件

## 总结与 2026 H2 规划

四大升级已落地：计算范式升级（Ray x Spark）、引擎能力升级（Xpark + Ray Libraries）、元数据系统升级（TCLake）、Agentic 能力升级（MCP x Skills x CLI）。

2026 H2 规划：① 产品持续演进（四大升级持续迭代）；② 客户共建验证（与灯塔客户在生产环境验证价值闭环）；③ 数据核心资产（探索 AI 时代数据作为核心资产的持续价值创造与变现路径）；④ 开放生态共建（Open Engine 战略，与开源生态共建共赢，让客户选最好用的引擎）。

## 相关页面

- [[tencent-ai-dlc-engines]] — AI DLC 三大核心引擎（TCRay/Xpark/Meson）深度解读
- [[ai-native-data-platform-vision]] — 大会开场演讲（AI 时代数据基础设施展望）
- [[workbuddy-data-practice]] — WorkBuddy 基于 AI DLC 的数据实践（产品演示中的 Agent 工具）
- [[lakehouse]] — 湖仓一体（DLC 数据计算所支撑的数据平台形态）
- [[agentic-data-cloud]] — Google Agentic Data Cloud（同类 AI 原生数据平台战略）
- [[databricks-2026-summit]] — Databricks 2026 峰会（大数据阵营的 Agent 化布局对照）
- [[apache-arrow]] — AI DLC 流水线中的数据交换层（WorkBuddy 实践中的 Spark→Arrow→Ray）
