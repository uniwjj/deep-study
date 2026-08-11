---
title: AI DLC 三大核心引擎解读
description: 腾讯云 AI DLC 三大引擎深度解读——TCRay（分布式计算引擎，生产级 Serverless Ray 企业增强）、Xpark（自研多模态计算引擎，50+ 多模态算子）、Meson（数据计算向量化引擎，1TB TPC-DS 3.6X）的背景、架构、关键能力与性能数据
aliases: [TCRay, Xpark, Meson, AI DLC 引擎, 多模态计算引擎, 向量化引擎]
tags: [big-data, ai-agent, tool, concept]
sources: [2026/08/11/AI Native数据湖让企业Agent拥有统一的智能数据底座（腾讯云专场）/3. AI DLC三大核心引擎解读.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# AI DLC 三大核心引擎解读

> DataFun Agentic AI Summit「AI Native数据湖让企业Agent拥有统一的智能数据底座（腾讯云专场）」专场演讲，对 AI DLC（见 [[tencent-ai-dlc]]）的三大核心引擎做深度解读：**TCRay**（分布式计算引擎）、**Xpark**（多模态计算引擎）、**Meson**（数据计算向量化引擎）。

## 总起：数据湖计算边界正在扩展

Agent 时代正在重塑数据平台的角色：数据湖不再是只作为服务人类分析决策的 ETL 与查询底座，而是需要升级为承载**多模态处理、模型后训练、批量推理、在线服务与 Agent 编排**的 AI Native 统一计算底座，让数据消费、轨迹沉淀、记忆生成与智能进化在同一基础设施上形成闭环。

传统 Workload vs AI Workload：大规模结构化数据处理 / 以 CPU 为核心 / 粗颗粒度调度 vs 海量多模态数据 / 以 GPU 为核心 / 资源细颗粒度调度——之间存在**范式鸿沟**（PyTorch、LLM、DeepSpeed 等 AI 生态）。

---

# 一、TCRay：AI DLC 分布式计算引擎

从多模态数据处理、模型训练到 Agentic AI 的统一计算底座。

## Ray 的核心定位

Ray is an AI compute engine——由一个核心分布式运行时（core distributed runtime）和一组加速 ML 工作负载的 AI Libraries 组成：

1. **Python 原生分布式底座**——贴近 Python 开发者生态，让 AI 应用可以用熟悉的 Python 方式规模化运行
2. **Task + Actor + Object 统一抽象**——用一套编程模型覆盖无状态并行计算与有状态服务，降低分布式开发门槛（Ray Core 支撑 Ray Data / Train / Tune / Serve）
3. **AI 全生命周期组件体系**——覆盖数据处理、模型训练、参数调优、推理服务等关键链路，支撑统一 AI Infra 建设

## Ray 产业趋势：从分布式运行时走向 AI Infra 标准组件

- **应用规模**：从开源热度走向大规模使用（GitHub Star 增长至 43.2k、PyPI 下载 6.9+ 亿次、贡献者 1500+）
- **后训练生态**：成为 RL/RLHF 框架的标准依赖（VeRL、OpenRLHF、Slime、AReaL、ROLL、NVIDIA-NeMo/RL）
- **社区治理**：从快速迭代走向企业级稳定——2025-10-22 成为 PyTorch Foundation Project，与 PyTorch、DeepSpeed、Safetensors、vLLM、Helion 并列
- **云厂商采纳**：从开源框架走向托管服务

## Ray 能力版图（基于 Ray Core）

- **数据域（Data Processing）**：数据预处理与清洗、多模态数据处理（图像/视频/音频）、Embedding 向量/索引构建、特征工程（生态：daft、pandas、Lance、Parquet）
- **训练域（Training）**：分布式训练、LLM 预训练、SFT 监督微调、强化对齐（DPO/RLHF）、超参搜索与实验并行（生态：PyTorch、DeepSpeed、XGBoost、TensorFlow、OpenRLHF）
- **推理域（Inference）**：离线批量推理、流式生成推理、模型即服务（Model as a Service）、多模型路由与级联（生态：vLLM、LMDeploy、Streaming）
- **生态域（Applications）**：RAG 检索增强、AI 智能体（Multi-Agents）、分布式推荐系统、多模态生成式应用（生态：LlamaIndex、Airflow、HuggingFace）

## 开源 Ray 的生产化局限：企业级托管、治理与云原生能力待增强

| 维度 | 开源 Ray 现状 | 生产平台需补齐 |
|------|-------------|--------------|
| 托管服务 | KubeRay 已提供集群与作业生命周期、自动伸缩等基础能力 | 统一作业入口、集群生命周期、环境与镜像、版本升级，让用户无需直接管理底层集群 |
| 稳定性保障 | Ray 已提供日志、指标和 Dashboard，可对接 Prometheus | 统一监控告警、历史作业追溯、故障诊断、自动恢复，形成可衡量的 SLA/SLO 保障 |
| 多租户治理 | Ray 的资源声明主要用于调度，并不等同于租户隔离 | 访问控制、配额与队列、优先级调度、资源隔离、租户用量统计，支持多团队共享算力 |
| 云产品集成 | KubeRay 已支持 Kubernetes 原生部署 | 打通云上网络（VPC/RDMA）、存储（COS/CFS/数据湖）、镜像和数据目录（TCCatalog），减少用户自行适配 |

## TCRay 能力架构：面向生产级 Serverless Ray 的企业增强

- **应用层**：离线作业（Batch Jobs：多模态数据处理[音/视/图/文]、特征工程、SFT & 强化学习、离线批量推理）+ 在线服务（Online Services：流式生产推理[对话/生成]、长周期任务异步推理[转写/长文档/视频]、向量检索服务）；生成式 AI 场景（RAG、NL2SQL、Text/Image/Video 多模态内容生成、Agentic Workflow 多智能体编排、Summarize/Extract 内容理解与摘要）
- **接入层**：Jupyter | VSCode | WebShell | CLI | SDK | REST API
- **框架**：Ray Data、Ray Serve
- **基础设施**：计算与容器（TKE/Kubernetes、Node Pool）、异构算力（CPU/GPU/TPU/NPU 及国产算力）、加速能力（TCQA）、网络底座（VPC/RDMA/…/NAT）、存储底座（COS、CFS）、数据湖 & Catalog（DLC Catalog / Hive Metastore）
- **运维与可观测**：Logs（日志关联检索）、Monitor（节点健康、CPU/GPU/内存、吞吐/时延、QPS）、Alerts（阈值、趋势、关键智能告警；稳定性阈值、故障恢复、资源池审计、过载保护与通知）、Profiling（性能剖析 CPU/GPU/IO/网络，算子级耗时、GPU Kernel、内存/显存占用，自动生成优化建议）、Tracing（Tensorboard/Nsight/Torch Profile/W&B/MLflow）

## TCRay 在 AI DLC 中的定位：连接数据湖与智能应用的 AI 计算平面

- **Data + AI Workloads 计算平面**：数据处理与 ETL（Meson、Daft、Spark——结构化/表数据处理）、多模态数据处理（Xpark、Ray Data、Data Juicer——非结构化/多模态）、传统机器学习、大模型训练 SFT（Ray Train、Metatron/Deepspeed/FSDP）、强化学习（veRL/OpenRLHF）、GenAI（Ray Serve、vLLM/SGlang）——统一构建在 **Ray Core 分布式运行时**（统一资源管理、任务调度、分布式执行；Actor/Task/Object）之上
- **数据湖与统一数据目录 TCCatalog**：统一元数据、统一权限管理、AI 数据资产支持、外部数据资产介入
- **存储**：结构化数据服务（流批一体 TCiceberg、Hudi、Delta）+ 非结构化数据服务（Lance 生态、Volume、Model）

## TCRay 建设进展与演进方向

建设进展：规模化 3k Nodes / 10w Actors；历史可观测（History Server 全面升级：Job 12h 内 12GB COS 存储、1339w Events、12w Tasks、13k 日志，Event 分类存储、Task 事件按需加载、动态 LRU 驱逐、Logs 增量上传、高优信息分级加载、无效信息剪枝降噪）；稳定性（Worker OOM 优化）；落地场景（数据科学、AI Coding、模型服务、运维 & 可观测）。

演进方向（面向规模化 AI Workload 的云上统一计算底座）：
1. 统一承载更丰富的 AI Workload，持续适配国产异构算力与开放生态（Workload + 场景 + 生态）
2. 端到端加速资源供给与任务执行，持续缩短作业从提交到完成的时间（快启动、快调度、快执行、快扩缩）
3. 将智能能力贯穿开发运维全流程，持续降低 Ray 的使用门槛（事前推荐、事中洞察、事后诊断）
4. 更省（空闲归零、低价资源、提升利用率，降低单位任务成本）

---

# 二、Xpark：AI DLC 多模态计算引擎

面向 AI 时代的腾讯云自研多模态计算引擎。

## 背景：数据处理从结构化计算走向多模态处理

AI 应用场景爆发，多模态数据快速增长：LAION-5B 公开图文数据集包含约 58.5 亿图文对；YouTube 每分钟上传超过 500 小时视频内容；全球数据量 2025 达 213 ZB、非结构化占比 92%、CAGR 21.4%；非结构化管理市场 92–285 亿美元、CAGR 15%（IDC 2025）。

传统大数据处理引擎在多模态场景下失配：数据形态复杂、算子体系不足、异构资源协同困难。Xpark 的解法：高性能的多模态算子、异构资源的高效协同、统一的多模态处理接口。

## 架构

- **多模场景**：LLM/VLM、RAG 知识库构建、训练数据准备、自动驾驶、具身智能、日志处理、用户画像、用户意图识别
- **接口层**：Xpark Dataset
- **多模计算**：Xpark 自研多模数据计算引擎（内置模型：TI-ONE、TokenHub、Embedding/CLIP、第三方模型），基于 TCRay
- **多模数据**：文本、向量、JSON、图片、PDF、PPT、视频；Iceberg、Paimon、Lance 表格式
- **存储**：ES/向量数据库、对象存储/CFS/TCLake Volume、TCLake Table
- **关键能力**：① 高性能多模态算子 ② 异构资源协同加速 ③ 分布式加速

## 关键能力

### 1. 多模态数据处理算子

问题与挑战：Spark 只具备比较基础的多模数据处理能力（如图片文件读取，复杂功能需适配 Python）；Ray 多模态算子较少/未经优化，部分性能比较差（如 read_videos）；异构流水线往往需要用户实现和调优，难度大。

解决方案：Xpark 内置丰富多模态算子库，优化高频场景——多模态算子（图片重采样、图片 OCR、图文匹配、PDF 解析、大规模文本去重、文本语言识别、音频转文本、视频场景切分）+ LLM AI Function（AI_SUMMARIZE、AI_SENTIMENT、AI_CLASSIFY、AI_FILTER）。

当前成果：**集成 50+ 多模态算子**，大幅降低开发成本；**通过模型级联，判别式算子相比单纯大 LLM 调用节约 70% LLM 请求**；支持 UDF，可快速迁移已有业务逻辑（如 Spark/Ray Data 的 read_images → resize(opencv) → map_batch(人脸模糊算子) → write_images）。

### 2. 异构资源协同加速

问题与挑战：CPU/GPU 速率不匹配导致流水线阻塞（GPU 可能等待 CPU 前置处理）；高带宽 + 分布式数据传输带来的「通信墙」瓶颈（多模态数据量大、分布式传输延时高）。

解决方案：CPU/GPU 流水线 Overlap（异步化封装预处理、模型推理、后处理）；减少跨机数据拷贝（CPU/GPU 协同调度 best-effort，Xpark Zero Copy + Ray Data 任务并行）。

当前成果（以 CLIP 图文匹配场景为例）：**推理吞吐相比开源 Data-Juicer 实现 3 倍提升**；**GPU 得到充分释放，使用率长期稳定接近 100%**。执行模型：CPU-based Ray Task（Download Image → Decode Image → Normalize Image → To Tensor）→ Ray Object Store（异步缓冲）→ GPU-based Ray Actor（CLIP Model：Image Encoder/Text Encoder，Compute Similarity：Dot Product/Cosine Similarity）→ 输出 Similarity Score。

### 3. 单机算子分布式化实现性能突破

问题：有状态单机算子往往较难实现分布式化，难以支撑大规模数据处理（如 text-dedup）。

解决方案：以文本精确去重为例——有状态、分阶段、分布式实现高性能去重；CPU-Intensive 关键路径使用 C++ 实现（BTS Union-Find 并查集，论文 IEEE 10598116），解决 Python 性能问题。流程：构图（Hash + Winnowing 分布式并查集）→ 数据切分 → 孤立文档过滤（无重复子串）→ 关联文档汇聚 → 按子图并行去重。

当前成果：**高性能 Exact Substring Dedup 算子**——比 text-dedup 开源单机算子快 47.8 倍（C++ 高性能分布式 BTS）；**高性能 Fuzzy Dedup 算子**——大幅领先开源 Data-Juicer（基于元数据唯一 ID 生成，省去一轮全量 IO；C++ 高性能分词器）。

## 典型 Pipeline

- **用户意图识别**：旧 Pipeline（ClickHouse 读取导出 parquet → 文本清洗获取 query → Embedding 开源模型生成向量 → 意图召回；手动分片、单机多线程、本地运维与调度）→ 新 Pipeline（Xpark + Ray：TCLake → Xpark 读取 → 文本清洗 UDF+MapBatches → DualEmbedding[开源模型 + 后训练学生模型]生成向量 → 向量拼接 Concat → 意图召回 Numpy UDF；原生 Xpark 算子、原生分布式、弹性集群、云上可观测[ TCRay Dashboard 可观测与统计]）
- **视频语义知识库构建**：视频数据（TCLake）→ 镜头切分 → 转码/抽帧过滤 → 字幕识别 → 水印检测 → 合成/动画检测 → 视频语义提取 → Embedding → 存入 TCLake Lance → 语义检索（客户使用）、语义去重（客户使用）

## 展望（时间线）

- 2025-H2：Xpark 立项，0.1.0~0.2.0 版本研发
- 2026-H1：0.3.0~0.5.0 版本研发
- 2026-07：视频处理算子丰富（美学评分、视频理解、场景切分）
- 2026-08 起：Xpark 优化器、VLA Trajectory Curation、Checkpoint、场景 Pipeline 覆盖与沉淀、开源筹备与发布

---

# 三、Meson：AI DLC 数据计算向量化引擎

为 AI DLC 打造的下一代计算内核。

## 背景与挑战

- **数据规模爆发式增长**：AI 与多模态带动计算负载持续增长，mixed workload 更复杂，近实时分析对算力吞吐与响应时延要求显著抬升
- **客户降本诉求迫切**：算力成本上行而预算趋紧，客户诉求从「减负」转向「以更优单位计算成本消化更多负载」
- **传统计算分析引擎的水土不服**：① 性能——传统计算引擎无法利用现代 CPU 的向量化和多核能力；② IO——云原生架构下对象存储带来高延迟；③ 全托管调优——高度依赖人工调参，试错成本极高，缺乏智能化的自适应调优机制

Meson 三件套：高性能向量化算子、并行执行模型、自适应优化。

## 技术架构

AI DLC 云产品（SQL、Spark、交互式分析）→ **Meson 高性能计算引擎**（计算引擎层：Pipeline 执行模型、向量化算子、自适应优化、列存内存格式；BSP 调度模型[离线批处理]、MPP 调度模型[在线分析]；弹性伸缩、资源池化）→ 统一存储（TCQA 数据加速服务：计算感知、智能分层、分层加速；TCLake Table：结构化流批一体、多模存储）。

## 核心能力

### 1. 向量化算子

现代 CPU 采用超标量流水线技术让单核 CPU 在同一时间执行多条指令，Meson 采用向量化技术提升指令运行吞吐量，压榨 CPU 的极限性能。

- **CPU 流水线空闲场景**：指令串行、流水线空闲——CPU 利用率 100% 但 IPC 仅 0.2（IPC: Instruction Per Clock）；指令并行、流水线打满——IPC 0.6，**吞吐提升 3X**
- **关联场景向量化优化**（哈希表关联）：常见原因（虚函数调用、分支预测失效、指令间存在数据依赖）→ 解法（拆解向量化单元、消除或减少分支、批量查找 batchFind 替代逐行 find）
- **Memory Stalled（内存停顿）**：原因（CPU Cache Miss、从主存加载需要 100ns+、引发 CPU 长时间阻塞）→ 存储层级：寄存器约 0.3ns/几十字节、L1 约 1ns/几十 KB、L2 约 6ns/几百 KB、L3 约 20ns/几十 MB、主存约 100ns/百 GB。Memory stalled 时 CPU 利用率 100% 但 Busy 态 <30%
- **聚合场景 Prefetch 异步访存优化**：预取未来第 16 个元素到 L1、内存控制器后台异步加载、CPU 和内存并行加载——理想效果 CPU 利用率 100%、Busy 态约 100%

### 2. Pipeline 执行模型

实现计算和数据并行解耦，增加并发能力。三代演进：传统 Volcano 模型（Row-Based 逐行交互、自顶向下 PULL 驱动、单并发执行）→ Vectorized 模型（Batch[4k~8k 行]粒度交互、消除解释执行成本，仍自顶向下 PULL 单并发）→ **Pipeline 模型**（Batch[4k~8k 行]粒度交互、自底向上 Push、多并发执行、拆分 CPU/IO Pipeline，计算与数据并行解耦）。

### 3. 高性能 IO

解决存算分离瓶颈，重塑数据访问效能。

- **元数据侧**：无效数据访问带来带宽浪费（基于元数据可提前裁剪数据）、元数据访问本身构成带宽瓶颈（海量分区/文件数使元数据请求量膨胀）→ 数据裁剪（利用元数据精准跳过无效数据块）、元数据缓存（statistics cache → optimizer、file meta cache → plan，加速文件寻址与优化器决策）
- **数据访问侧**：等待 IO 完成导致 CPU 执行卡顿、对象存储的高延迟导致查询延迟 → IO 异步和 prefetch（让 CPU 和 IO 并行化）、IO 合并和数据缓存（merge io request、memory and disk cache，最大化避免低效的对象存储访问）

### 4. 自适应能力（TCinsight 智能调优）

解决全托管调优成本：静态配置难以适配多变的 OLAP 负载；全托管场景下人工调优成本过高、经验难以沉淀。

TCinsight 基于传统 ML 模型，实现系统自动检测、查询分析和调优，无需人为干预：① 动态调整——运行时优化并发度（大宽表、高压缩率、小文件）与 Shuffle 分区数；② 智能决策——自动切换 Join 策略（HashJoin、SortMergeJoin）与 Broadcast Join；③ 实时优化——adaptive partial agg。

智能性能调优效果：Bad plan → Good plan（如 shuffle 1000 分区 → 细分），调优人效下降（9 人天 → 4 人天）。图中具体提升倍数/百分比数字过小，未能可靠辨认，此处不列。

## 性能与收益

**1TB TPC-DS 性能提升 3.6X，CPU 消耗下降，由 CPU 变为 IO 瓶颈**：

- Meson TPC-DS 1TB 相较社区 Spark 整体性能提升 **3.6X**；q23/q24/q64/q78 等计算密集型查询提升 **5X**
- 在整体性能提升 3.6X 的情况下，Meson Spark CPU 消耗下降，集群 IO 吞吐提升近 1X——向量化提升 CPU 效率显著，**瓶颈点由 CPU 转向 IO**
- 生产实测：Meson Spark 引擎所有集群的 CPU 平均使用率 40% vs 社区 Spark 80%

场景级对比（Meson vs Spark）：

| 场景 | 提升 | 场景描述 | 优化点 |
|------|------|---------|--------|
| Join | 10X | dim join 大表 2 亿 7000 万 | 数据 layout 优化、算子算法优化、Prefetch 与访存优化 |
| Aggregate | 3.4X | 低聚合度、多列 3 亿 7000 万数据 | 数据 layout 优化、算子算法优化、Prefetch 与访存优化 |
| Shuffle | 3.6X | 2 亿 8 千万数据 200 分区 | 列式压缩、向量化分区 |
| Scan | 1.75X | 4 列 700 万数据 | 缓存（SSD Cache）、列式读取 |

（2026-07-25 演讲）

## 相关页面

- [[tencent-ai-dlc]] — AI DLC 产品发布（三大引擎是其引擎能力升级的组成部分）
- [[ai-native-data-platform-vision]] — AI 时代数据基础设施展望（AI DLC 新架构总览）
- [[lakehouse]] — 湖仓一体（Meson/TCRay 所承载的数据平台形态）
- [[apache-arrow]] — 内存交换层（Ray Data / Arrow 在 AI DLC 流水线中的角色）
- [[workbuddy-data-practice]] — WorkBuddy 基于 AI DLC 的数据实践（Xpark 多模态引擎的真实使用方）
- [[apache-ossie]] — 开放数据标准栈（Lance 多模态表格式在数据湖层的定位）
