---
title: 下一代 ODPS：全模态引擎和 Agentic 全面升级
description: 阿里云 MaxCompute（ODPS）2026 年 8 月 Agent 大会演讲——全模态存储（Blob）+ 模型统一管理 + CU/GU/Token 异构算力 + SQL/MaxFrame 双引擎 AI Function，以及面向 Agent 时代的 MCMCP、MaxAgent 与分布式 AI 计算引擎成效
aliases: [下一代ODPS, ODPS 全模态引擎, MaxCompute Agentic, ODPS Agentic]
tags: [big-data, ai-agent, platform, tool]
sources: [2026/08/11/Agent 驱动的数据智能新范式(阿里云专场)论坛/01-赵弘扬-下一代ODPS：全模态引擎和Agentic全面升级.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 下一代 ODPS：全模态引擎和 Agentic 全面升级

2026-08-11 2026 Agent 大会「Agent 驱动的数据智能新范式（阿里云专场）论坛」演讲。演讲人：**赵弘扬**｜阿里云。主题：MaxCompute（原名 ODPS）从大数据引擎升级为"全模态引擎 + Agentic"——Data+AI 能力全面升级（模型、算力、存储、AI Function），并通过 MCMCP/Skills 把底层能力封装成 Agent 可调用层。

## 摄取说明

PDF 共 20 页，已用文本层提取 + 关键页（P12/P14/P19）转图 OCR 视觉复核，架构图与数字均出自原文；OCR 无法完全辨认的小字未摄取。

## 逐页索引

| 页码 | 主题 | 已摄取信息 |
|------|------|------------|
| P1 | 封面 | 下一代ODPS：全模态引擎和Agentic全面升级；演讲嘉宾：赵弘扬 |
| P2 | ODPS 跟随时代进程的成长路径 | 大数据技术四期演进（探索期/发展期/普惠期/变革期）；阿里云与全球大数据里程碑时间线 |
| P3 | ODPS 曾获得的重大成就 | IDC 中国公有云大数据平台市占率第一（2023，连续四年第一）；中国数据治理市占率第一（2023，连续三年第一）；影响力/国家奖项 |
| P4 | 分节 01 Data+AI 能力全面升级 | 阿里云 MaxCompute（原名 ODPS） |
| P5 | Data+AI 能力全面升级 | AI Function 推理、异构资源灵活调度、多种模型覆盖全场景、全模态存储四大能力 |
| P6 | 模型层 | 公共模型/导入模型/远端模型/用户训练模型四种模型类型，SQL & MaxFrame AI Function 无缝调用 |
| P7 | 算力层 | CU Quota（CPU）/GU Quota（GPU）/Inference Quota（Token）三种算力与计费方式、适用模型 |
| P8 | 存储层 | Blob 数据类型与四大多模态场景（AI 训练数据集管理/多模态数据处理流水线/多模态加工缓存层/非结构化数据入湖） |
| P9 | AI Function | SQL AI Function 9 个函数 + MaxFrame AI Function；选择 MC AI Function 的 6 大理由 |
| P10 | AI Function 核心优势 | 深度集成百炼、双引擎统一对接、精细计费与管控 |
| P11 | 分节 02 全模态案例场景 | 阿里云 MaxCompute（原名 ODPS） |
| P12 | 汽车行业方案：智驾数据处理产线 | 具身行业 Skills（阿里云门户 Skills 热门榜第 1 名）；产线分层架构与角色 |
| P13 | 具身智能 EgoCentric | CPU+GPU 混合算力、可断点续跑；人手视频 → LeRobot 数据集全流程自动化 |
| P14 | 分布式 AI 计算引擎 | 日处理 100 万+ 图片及 50 亿+ Token；性能提升 2 倍以上；周期从周级缩短至日级 |
| P15 | 分节 03 Empowering Digital Workers | 阿里云 MaxCompute（原名 ODPS） |
| P16 | Agentic 时代接入方式 | 用户 Agent 接入、MCMCP 服务、安全体系、计算/存储能力、AutoMV 与 Delta Table 防 Agent 破坏 |
| P17 | MaxAgent 运营助手 | 数据开发日常运维/资源管理员移动运维/降本增效/智能问数四大场景 |
| P18 | MaxCompute MCP | MCMCP 方法分组、生产就绪五要点、配套 Skills |
| P19 | AI 数据探索客户端 | MaxMan AI 桌面级客户端 + Studio：SQL 生成/优化、多模态数据管理、Data Agent 对话 |
| P20 | 结束 | THANKS 演讲嘉宾：赵弘扬 |

## 演进历程与成就

### 跟随时代进程的成长路径

大数据技术四期演进：

- **探索期**：互联网时代到来，以分布式调度、存储为核心的基础设施建设时期，打破数据库以低成本大规模扩张问题。分布式计算模型为 MapReduce
- **发展期**：开始关注开发效率，分布式计算模型针对场景细分，总体向 SQL 靠拢
- **普惠期**：开始关注投入企业生产必须的能力：工作流、安全、治理、规模、稳定性等，出现数据中台概念
- **变革期**：开始关注 Bigdata+AI 的异构计算能力，以及对多模态海量数据的高效处理能力

阿里云大数据里程碑（时间线按页面布局）：2001 年 Alibaba Apsara / MaxCompute（ODPS）写下第一行代码（基于 RDBMS：Oracle & Greenplum，当时亚洲最大的 Oracle Rack）→ 2009 年 MaxCompute 公共云售卖单集群 5000 台 → 2013 年 MaxCompute 1.0，MaxCompute 和 DataWorks 构建完整阿里数据中台体系 → 2015 年 MaxCompute 2.0 单集群过万台 → 2018 年 MaxCompute 3.0 综合成本降低 30%、发布 Hadoop 联邦计算 → 2021 年 MaxCompute LakeHouse 2.0 → 2023 年 MaxCompute Data+AI 计算引擎正式发布 → 2024 年 MaxCompute 全面进入 AI 异构计算时代 → 2025 年 MaxCompute 分布式 Python 计算框架首次发布。全球大数据市场里程碑（同页时间线，部分对应）：1990 数据仓库出现、2003 GFS、2004 MapReduce、2006 Hadoop 第一个版本发布、2010-2012 Hive/Flink/Spark、2014 Google BigQuery/AWS Redshift、2018 AWS Lake Formation、2019 DeltaLake/Hudi/Iceberg、2020 Snowflake、2021 StarRocks/Apache Paimon、2024 BigQuery Cortex AISQL、2025 Snowflake AI Function with Generative AI 等。

### 曾获得的重大成就

- IDC：中国公有云大数据平台市场占有率第一（2023 年），连续四年排名第一
- IDC：中国数据治理市场占有率第一（2023 年），连续三年排名第一（2023 年中国数据治理市场：整体规模 ¥2934.5M、+9.1%；阿里云份额 ¥902.60、同比 16.8%，单位按页脚注释为 RMB Million 与增长率 %）
- 影响力（亚洲）领先：2024 年信通院数据智能平台专项测试认证；2022 年阿里云自主研发的大数据智能计算平台 ODPS 入选世界互联网领先科技成果；2021 年 Forrester CDW 2021 Wave，MC+DW 全球卓越表现者象限，亚洲唯一入选云厂商；2021 年 Gartner DSML（数据客户与机器学习），第一年入榜，国内唯一；2020 年 Gartner 云数据库/数据分析国内唯一进入 Leader 象限的云厂商
- 国家奖项：2017 年中国电子学会科技进步特等奖（被业界誉为"科技界金棕榈"）；2018 年浙江省科技进步一等奖；2018 年国家大数据博览会十佳产品最佳案例实践奖；2018 年中国数字化转型与创新案例大会年度大数据创新产品奖；2019 年大数据"星河（Galaxy）"奖最佳大数据产品奖（TOP10）

## Data+AI 能力全面升级

四大能力支柱：

- **AI Function 推理**：将复杂的 AI 推理场景"降维"成一行简单代码、SQL 和 MaxFrame 双引擎开发生态支持，能力丰富、对接简单，构建面向智能体时代的统一开发与运行范式，释放数据价值
- **异构资源灵活调度**：支持多种不同类型计算资源（如 CPU、GPU、PPU 等），支持多种不同类型计费方式（按数据扫描量、CU、GU、Token 等）；根据作业任务和资源特性，智能地将任务分配到最合适的资源上执行，以实现性能最优、成本最低或效率最高的目标
- **多种模型覆盖全场景**：MC 支持统一模型对象管理并提供开箱即用的公共模型，可直接对接百炼各类商业化大模型，同时通过无缝集成数据处理与 AI 能力，完成从数据准备、数据处理、到模型推理的完整流程，降低企业级大模型应用门槛
- **全模态存储**：以多模态数据为核心，深度对接 DLF，支持原生多模态数据存储格式、统一数据访问接口，实现元数据统一管理，同时面向数据处理/推理场景深度优化

### 模型层：四种模型类型统一管理

| 类型 | 说明 |
|------|------|
| 公共模型 | 内置开箱即用的百炼商业化模型及 Qwen 3、Deepseek 系列开源大模型，预先创建在 BIGDATA_PUBLIC_MODELSET 公共项目 |
| 导入模型 | 用户导入在外部训练调优后保存的自定义模型文件，指定 OSS 模型文件地址，并注册至 MaxCompute |
| 远端模型 | 对接 PAI-EAS 上已经部署好的模型，指定对应访问 Endpoint 和 token，注册成为 MaxCompute 远端模型 |
| 用户训练模型 | 用户进行传统机器学习模型训练，执行 to_odps_model API 将其保存成为 MaxCompute 模型 |

同时提供模型和模型版本能力，方便模型版本管理和灰度验证；支持完整的模型元信息与权限体系；基于 SQL & Python（MaxFrame）支持模型的 CRUD；MaxCompute 控制台支持模型的白屏化查看；支持 SQL & MaxFrame AI Function 无缝调用。说明：百炼模型在 MC 中属于公共模型类型，用户开箱即用，不需要创建或购买百炼资源和配置 API-KEY，由 MC 提供全托管的模型能力。

### 算力层：CU / GU / Token 三种资源

| 资源类型 | 资源优势 | 适用/推荐模型 | 业务场景 |
|----------|---------|--------------|---------|
| CU Quota（通用 CPU 资源） | 海量资源，支撑超大规模（数十万核）并发计算 | 小尺寸模型（<=8B）；推荐 Qwen-0.6B/4B、Deepseek-R1-1.5B/7B | 开发测试场景；小规模数据量推理任务（性价比极高） |
| GU Quota（GPU 智算资源） | 支持用户独享 | 大尺寸模型（>=8B）；推荐 Qwen3-14B、Deepseek-R1-14B | 对推理效率与吞吐有极高要求的大模型作业（如海量语言翻译、结构化提取） |
| Inference Quota（按量付费 Token 资源） | 按模型实际 Token 消耗计费，灵活可控 | 百炼商业化大模型；推荐 Qwen3-max、Qwen3.6-plus、Text-embedding-v4 | 希望开箱即用、免底层资源运维；需要顶级模型质量的复杂分析；按使用量付费、无需较高成本做资源预留 |

### 存储层：Blob 多模态存储

Blob（Binary Large Object）数据类型是 MaxCompute 提供的一种用于在表中存储图片、音频、视频、文档等非结构化二进制大对象的数据类型。通过 Blob 类型，可以将多模态数据的原始文件、元信息和标注信息统一存储在同一张 MaxCompute 表中，使用 SQL 统一查询和维护，通过 MaxFrame 和 SQL UDF 批量加工处理。

四大场景：

1. **AI 训练数据集管理**：将图片、音频、视频与标注标签存储在同一张表中，通过 SQL 过滤所需样本数据集
2. **多模态数据处理流水线**：视频切帧、音频转文本、内容打标等多步骤流水线的中间结果和最终产物统一存储
3. **多模态加工缓存层**：作为对象存储等数据源的缓存层，提升 MaxFrame、SQL 大规模并行计算的吞吐性能，解决小文件批量请求的 QPS 限制
4. **非结构化数据入湖**：将分散在外部存储中的文件统一导入 MaxCompute，扩展元信息建立企业非结构化资产库，获得事务保障和统一权限管控

### AI Function：SQL 和 MaxFrame 双引擎统一对接

MC AI Function 能帮助用户摒弃繁琐低效的数据搬运过程，实现"数据即模型输入、结果即模型输出"，构建 Data+AI 开发新范式。

- **SQL AI Function（兼容 SQL 生态）**：AI_GENERATE、AI_CLASSIFY、AI_EXTRACT、AI_SIMILARITY、AI_SENTIMENT、AI_SUMMARIZE、AI_TRANSLATE、AI_EMBEDDING、ML_PREDICT
- **MaxFrame AI Function（兼容 PYTHON 生态）**：generate 接口、task 接口

选择 MC AI Function 的 6 大理由：

1. **极简低代码开发**：一行 SQL/Python 函数即插即用，零部署成本，大幅缩短开发周期
2. **多引擎全面支持**：SQL 与 Python（MaxFrame）双管齐下，兼容 Pandas 风格 API
3. **企业级无缝集成**：与 MaxCompute 的模型、计算资源、权限体系完美融合，对接简单
4. **极致高性能扩展**：支持分布式并发执行，可支撑单次 PB 级数据处理规模，线性扩展
5. **支持大规模生产**：支持自动并发切分、worker 内 sleep 机制、限流控制，保障作业稳定性
6. **灵活多维计费**：完美适配各种业务场景，无缝对接 MC CU/GU/Token 多样化资源池

核心优势：深度集成百炼赋能大模型推理（一行 SQL 即可完成 PB 级数据量大模型推理工作，对数仓工程师零学习成本，可将模型推理完美融入现有数据 ETL 工作流）；双引擎统一对接（全面兼容 Python 生态，算法工程师可直接调用 MaxFrame SDK 接口，实现数据处理与推理在库计算）；精细计费与管控（独立 Quota 方式管理模型推理资源，实现模型计算资源与 CU 等基础计算资源统一定义即开即用；Token 计费模式按量计费成本可控）；全模态支持（原生支持结构化数据与文本、图像、音视频等多模态数据联合分析与向量化）；端到端数据闭环（在同一框架内完成数据清洗、预处理、模型推理、打标的完整数据开发链路，降低数据搬迁成本）。

## 全模态案例场景

### 汽车行业方案：智驾数据处理产线

推出具身行业 Skills：支持通过 AI Function + 百炼大模型进行行业数据打标与向量化全流程，一句话即可生成完整作业，拿下阿里云门户 Skills 热门榜第 1 名。

产线架构（分层，OCR 复核）：多源数据输入（文件上传、人工标注数据、标签/属性信息）→ 数据接入通道（DataHub/Kafka，视频图像文件、视频/图片/元数据）→ 数据中台层（DataWorks + MaxCompute，Flink 实时/离线计算引擎，MaxCompute 统一计算引擎）→ 存储索引层（统一数据湖仓：OSS 样本库，原始数据/中间数据/样本文件）→ 业务产出层（PAI 模型训练：模型训练/微调/评估；MaxFrame 核心引擎：数据预处理（Wav2Vec 等语音增强类算子）/时空对齐（时间对齐/空间配准）；Hologres 高性能分析数据库：结构化数据/标签数据；向量数据库（Milvus/Elasticsearch）：相似检索；业务决策（BI/大屏）：数据分析/可视化/监控）。配套元数据管理（数据血缘/版本/质量）与工作流编排（DAG 任务调度）。

使用角色：数据研发工程师、AI 算法工程师、分析人员、数据标注团队（提供高质量标注数据）、数据工程师（负责数据接入与稳定传输）、数据研发团队（负责数据开发与计算任务）、数据平台运维（负责存储与索引管理）、业务团队（使用数据驱动业务决策）。

### 具身智能 EgoCentric

MaxCompute | CPU+GPU 混合算力 | 可断点续跑。EgoCentric Pipeline 端到端自动化：从原始视频到可训练数据集，全流程无需人工干预。

流水线：人手视频 → 切分 → 抽帧 → 深度估计 → 3D 手部+位姿 → 动作分割 → 语义标注 → LeRobot 数据集（资源依次为 CPU 资源 → CPU 资源 → GPU 资源 → GPU 资源 → 大模型资源 Token → 大模型资源 Token）。

工程化挑战与解决方案：

- 容错不中断：单片段失败只记录不阻塞
- GPU 显存复用：HaWoR 各阶段顺序执行复用显存
- OSS 挂载：避免多 FUSE 写入冲突
- 断点续跑：每阶段独立 MaxCompute 表，可从中间恢复

输出格式：LeRobot v2.0 标准格式——data/ Parquet（state 8D + action 7D）；videos/ MP4（原子动作视频切片）；meta/ JSONL（语言指令）；HDF5；兼容 pi0 / GR00T / OpenVLA。

### 分布式 AI 计算引擎成效

赋能超大规模自动化数据处理产线：模型接入内置集成阿里云百炼，支持 Qwen/DeepSeek 等模型，涵盖分类、打标、向量化，无需自建推理服务；稳定性保障为全自动容错机制、TPM/RPM 限流适配、Worker Sleep，确保任务零人工干预；DataWorks 统一调度、数据流统一，端到端自动化、无数据搬运。

核心成效指标：

- **亿级规模**：日处理 100 万+ 图片及 50 亿+ Token
- **性能提升 2 倍以上**（Before/After）
- **周期压缩**：从"周级"缩短至"日级"

## Agentic 接入方式：数字员工的统一能力接入层

MaxCompute 的专属 Agent 可以满足 AiQuery、AiOps 等应用场景，同时提供 MCMCP 与 Skills 结合的方式，将底层元数据、计算和存储服务封装成 Agent 可理解、可执行的统一能力接入层，让企业级客户可以通过 Agent 完成超大规模数据分析、高性价比多模数据加工处理和智能化运维。

- **用户 Agent 接入**：OpenClaw、Dataworks Agent、Qwen Code、QoderWork
- **MC Skills**：Agent 通用，包含常用命令、开发模板、使用限制等 → MCMCP 服务（Oauth 认证）：OpenAPI / StorageAPI / CatalogAPI 封装成可被 Agent 直接调用的结构化工具
- **Metadata + Semantic**：Catalog / Schema / Table / Partition
- **安全体系**：授权、审计、数据发现、敏感等级、脱敏、行级权限、数据共享
- **Compute Engines**：MC SQL、MaxFrame、MC Spark；异构算力 CU/GU、AI Function、模型
- **Storage**：Append / PK Delta Table、Blob / JSON / ARRAY 等数据类型；冷热分层、多副本、多 AZ 容灾、回收站/Time Travel、数据快照、存储加密
- **AutoMV**：无惧 Agent 重复查询，自动物化，空间换时间降算力
- **Delta Table**：无惧 Agent 破坏数据，TimeTravel 随时回滚
- 专属入口：**MaxAgent 运管助手**（覆盖作业诊断、权限排查、成本优化等运维场景）；**MaxCompute AI 数据探索客户端**（桌面级 Agent + Studio 客户端）

### MaxAgent 运营助手

覆盖资源管理、成本治理、业务分析全链路。免控制台登录、移动端操控闭环，打通钉钉、飞书、微信机器人通道，实现"聊天即服务"的交互。四大场景：

| 场景 | 痛点 | MaxAgent 能力 |
|------|------|--------------|
| 01 数据开发日常运维 | 作业失败/慢，排查耗时长；查询性能瓶颈，不知如何优化；全表扫描浪费资源 | 作业运维与智能诊断；物化视图推荐与收益评估；Auto Cluster 聚簇优化 |
| 02 资源管理员移动运维 | 项目配置繁琐，参数易遗漏；Quota 利用率不明，扩容凭感觉；外出告警无法及时处理 | 项目配置与参数管理；Quota/路由/计划/分时配置；机器人移动端闭环处置 |
| 03 降本增效 | 费用增长不知原因；存储大户不清，优化无方向；优化手段单一，收益不可量化 | 成本智能分析与趋势；计量成本归因与 Top 消费者；存储观测与分层推荐 |
| 04 智能问数 | 不懂 SQL，依赖数据团队；无法定位和识别目标表和数据业务逻辑；日常报表手动制作 | NL2SQL 自然语言查数据；上下文追问深度分析；跨域联动：查询+诊断 |

### MaxCompute MCP（MCMCP）

MaxCompute 提供的 Model Context Protocol（MCP）服务，用户可以使用丰富的 Agent 生态配置接入 MCMCP，借助配套的 Skills 用户可通过自然语言对话方式管理和分析 PB、EB 级数据。

方法分组：身份与权限确认（check_access）；元数据（list projects / schemas / tables / partitions）；Knowledge Catalog（search_meta_data）；分析计算（cost_sql、execute_sql、get_instance_status、get_instance）；数据管理（create_table、insert_values）。

生产就绪、安全、Agent 友好：

1. 丰富的认证方式，支持 AK/SK、STS、Credentials URI 等无 AK 方式
2. 元数据搜索服务，帮用户自然语言找表
3. 执行 SQL 可以约束仅允许只读查询
4. 执行 SQL 前可以走成本评估审批
5. 建表时提供表属性选项（生命周期、主键、部分列更新等）

配套 Skills：Sql-Generation、Maxframe-Coding、Information-Schema、Project-Qouta-Manage。

### MaxCompute AI 数据探索客户端（MaxMan AI）

桌面级生产力 + Data Agent + 多模态数据管理。客户端 + Studio 特性（界面 OCR 复核）：多 project 树（杭州 project、新加坡 project 等，Catalog 多 project）；SQL Editor 智能补全（基于元数据）；自然语言生成 SQL（如"按 region x category 透视并追加环比 SQL"）；SQL 优化建议（分区过滤前置，扫描 4.5GB > 320MB；聚合字段显式命名便于图表识别；DESC/SHOW 等 MC 命令结果统一呈现；一键插入原文件）；本地文件快速转 Table（Excel/CSV/Parquet/JSON Lines）；AI 推断 DDL → 一键建表 → 任务进度追踪；可视化/透视/质量检查（如空值 0.8%、极值通过）；Logview DAG 分析（CPU 338 cores、耗时 10.3s）；MaxMan AI Data Agent 工具（list_tables、desc_table、run_sql）；模型可选 Qwen3-Max、DeepSeek、Claude（界面可切换）。

## 相关页面

- [[maxcompute-skills]] — MaxCompute MCMCP 与 Skills 体系（2026-05 虾聊日）
- [[maxcompute-data-ai]] — MaxCompute Data+AI 演进（BLOB/Object Table/MaxFrame）
- [[hologres-skills]] — Hologres CLI/Skills/AI Function
- [[dataworks-data-agent]] — DataWorks Data Agent（DataWorks Agent 接入 ODPS 生态）
- [[ai-native-data-platform-report]] — AI 原生数据平台研究报告
- [[open-data-stack-evolution]] — 开放数据标准栈演进
- [[agentic-data-cloud]] — Google Agentic Data Cloud（另一家的 Agent 数据平台范式）
