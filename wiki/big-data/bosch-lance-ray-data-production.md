---
title: 博世智驾训练数据 Lance/Ray 生产实践
description: 博世智驾（ADAS）训练数据生产实战（英文演讲）——Lance 作为数据中心统一多模态数据集（Dense+sparse 一体化、版本化可训练交付）、Ray 作为任务中心（本地验证→集群执行→透明调度→快速聚合），由模型问题驱动的数据生产飞轮闭环
aliases: [Bosch AD Data Production with Lance/Ray, Lance + Ray, 数据中心+任务中心, Model-Driven Flywheel, 博世智驾训练数据]
tags: [big-data, ai-ml, practice]
sources: [2026/08/11/AI Native数据湖让企业Agent拥有统一的智能数据底座（腾讯云专场）/5. Bosch_AD_Data_Production_with_Lance_Ray.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 博世智驾训练数据 Lance/Ray 生产实践

> DataFun Agentic AI Summit「AI Native数据湖让企业Agent拥有统一的智能数据底座（腾讯云专场）」博世英文演讲：**Bosch ADAS Training Data in Practice——From multimodal complexity to a scalable data flywheel**（从多模态复杂性到可扩展的数据飞轮），主题为 **DATA CENTER x TASK CENTER（数据中心 × 任务中心），Practice in Tencent Cloud**。配套中文实践演讲见 [[bosch-ray-hybrid-computing]]。

## 三个问题与一套数据底座（01 / What We Need to Solve）

核心判断：**高效使用数据比孤立的存储优化更重要（Efficient data use matters more than isolated storage optimization）**。

1. **统一多模态数据（UNIFIED MULTIMODAL DATA）**——Dense 与 sparse 数据必须共同管理，而不是变成相互割裂的独立系统
2. **高效使用数据（EFFICIENT DATA USE）**——数据应该以更少的复制、交接和对齐步骤变成 train-ready（可训练）状态，让数据交付后更快进入训练
3. **模型驱动的数据飞轮（MODEL-DRIVEN FLYWHEEL）**——模型失败（问题）应当生成下一轮数据生产任务和目标数据集

## 目标架构（02 / Target Architecture）

**Lance + Ray: Data Centralization Meets Task Centralization**——数据中心 + 任务中心，统一支撑数据生产。一个共享底座连接生产、训练与在线评测：

- **LANCE - DATA CENTER（数据中心）**：一个逻辑数据层统一管理多模态数据集、版本及训练访问（One logical data layer manages multimodal datasets, versions and training access）
- **RAY - TASK CENTER（任务中心）**：一个执行层统一管理任务提交、资源、状态及结果聚合（One execution layer manages task submission, resources, status and aggregation）
- 闭环：**DATA → TASKS → MODEL FEEDBACK**

## Lance 数据中心（03 / Lance Data Center）

**Manage Dense and Sparse Data as One Multimodal Dataset**——Dense 与 sparse 共同构成一个可训练数据集，交付物是 train-ready 数据集而不是又一个中间交接物：

- **DENSE DATA（稠密数据）**：图像、视频和点云始终关联在同一数据集上下文中
- **SPARSE DATA（稀疏数据）**：标签、元数据和模型输出共享身份、版本与血缘（identity, version and lineage）
- **TRAINING ACCESS（训练访问）**：交付后的数据集可以直接被训练和评测消费

**ONE DATASET · ONE VERSION · ONE PATH TO TRAINING**（一个数据集、一个版本、一条通往训练的路）。参考 Lance 文档：不可变数据集版本与随机访问（immutable dataset versions and random access）。

## Ray 任务中心（04 / Ray Task Center）

**Move from Local Validation to Cluster Execution with One Task Model**——一套任务模型，从本地验证扩展到集群执行；资源请求、执行状态和结果变得可见且一致：

1. **VALIDATE LOCALLY 本地验证**——local 模式验证逻辑并评估资源开销
2. **SUBMIT ONCE 一次提交**——将验证完成的任务直接提交到共享集群
3. **EXECUTE TRANSPARENTLY 透明执行**——按明确资源需求调度细粒度任务
4. **AGGREGATE FAST 快速聚合**——聚合分布式结果并快速交付业务方

参考 Ray 文档：local-to-cluster execution 与 resource-aware scheduling。

## 价值

### 对基础设施（05 / Value for Infra）：统一底座减少重复建设与长期维护成本

- **ONE DATA FOUNDATION 统一数据底座**——共享的多模态数据模型取代 dense + sparse 两套独立体系
- **ONE TASK FOUNDATION 统一任务底座**——共享的执行模型取代业务自建的调度器和队列
- **LOWER MAINTENANCE 降低维护成本**——通用接口、可观测性和运维方式降低长期成本

**STANDARDIZE THE BOTTOM LAYER; KEEP BUSINESS LOGIC FLEXIBLE**（底层能力标准化，保持灵活的业务逻辑）。

### 对数据生产（06 / Value for Data Production）：透明流程让数据工作成为可管理的生产线

- **TRANSPARENT PROCESS 过程透明**——每个生产阶段有可见的输入、输出、状态和责任人
- **FINER GRANULARITY 粒度更细**——任务和资源按可高效调度的单元分解
- **CENTRALIZED DATA 数据集中**——中间产物和最终产物保持连接在同一受管数据集中
- **HIGHER EFFICIENCY 效率更高**——更少的等待和对齐缩短从需求到训练的路径

## 业务场景（07 / Business Scenario）：模型问题驱动的数据生产闭环

**Model-Driven Data Production Loop**——从模型缺陷到可训练数据集与在线评测的端到端闭环（六步）：

1. **MODEL GAP 模型问题**——将失败案例转化为数据需求
2. **DATA MINING 数据挖掘**——发现目标场景与候选样本
3. **AUTO-LABELING 自动标注**——规模化生成初始标签
4. **HUMAN REPAIR 人工修复**——修复困难及长尾场景
5. **TRAIN-READY DATASET 可训练数据集**——对合格数据进行版本化交付
6. **TRAIN & EVALUATE 训练与评测**——训练、评测并反馈新问题（回到第 1 步）

支撑体系：DATA CENTER = LANCE | TASK CENTER = RAY。

## 场景对比

### Ray 场景（08 / Ray Scenario）：Mining Workloads——从分散任务调度到透明的集群执行

- **BEFORE**：调度模式不统一、资源开销各自评估、任务经常排队等待——结果：交付压力大（high delivery pressure）
- **NOW WITH RAY**：本地验证资源、提交同一作业到集群、在统一透明的流程中聚合结果——结果：更快更可预测的交付（faster and more predictable delivery）

### Lance 场景（09 / Lance Scenario）：数据生产——从湖仓交接（Lakehouse Handoff）到可训练交付

- **BEFORE**：挖掘 → 自动标注 → 人工修复 → 湖仓（lakehouse）→ 下游统计 → 训练对齐——结果：训练前仍需多次交接和重新对齐（many handoffs before training）
- **NOW WITH LANCE**：挖掘 → 标注 → 修复 → 可训练 Lance 数据集 → 训练 → 在线评测——结果：**数据达到交付标准即可启动训练（delivery means training can start）**；训练和在线评测成为生产数据集的直接消费者

## TAKEAWAY（10 / Takeaway）

**Let Model Problems Drive Data Production（模型问题驱动数据生产）**：

- **LANCE**：集中管理多模态数据，交付可直接训练的数据集
- **RAY**：集中管理任务，让本地验证的作业平滑扩展到集群
- **FLYWHEEL**：以模型反馈连接数据生产、模型训练与在线评测

**ONE DATA CENTER · ONE TASK CENTER · ONE CONTINUOUS LOOP**（一个数据中心、一个任务中心、一个持续循环）。

## 相关页面

- [[bosch-ray-hybrid-computing]] — 博世同大会中文演讲（Ray on TKE 混合计算工程落地细节）
- [[tencent-ai-dlc]] — 腾讯云 AI DLC（Lance 是其统一存储层「多模态 Lakehouse 存储」的组成部分）
- [[tencent-ai-dlc-engines]] — TCRay/Xpark 引擎（Ray 能力版图与 Lance 生态）
- [[lakehouse]] — 湖仓一体（Lance 场景中「湖仓交接」的对照对象）
- [[apache-ossie]] — 开放数据标准栈（Lance 在五层标准栈中的定位）
- [[ai-native-data-platform-vision]] — AI 时代数据基础设施展望（数据即「认知燃料」）
