---
ingested: 2026-05-09
wiki_pages: [big-data/maxcompute-data-ai]
---

## MaxCompute Data+AI：大数据平台的 AI 化演进与实践

**文章信息**：
- **原文标题**：MaxCompute Data+AI：大数据平台的 AI 化演进与实践
- **来源**：https://mp.weixin.qq.com/s/D-VRouihJ7RButFJ90WLNA
- **作者**：刘洋（阿里云智能集团 产品专家）
- **发表时间**：2026-04-27
- **出品社区**：DataFun

---

**一段话总结**：

阿里云产品专家刘洋分享了 MaxCompute 在 Data+AI 方向的核心演进。MaxCompute 作为阿里云核心大数据计算平台，构建了从存储到计算的全链路 AI 能力：存储层通过 BLOB 类型和 Object Table 实现多模态数据（图片/视频/音频）的统一管理，支持跨 OSS、Hologres 的元数据统一访问；计算层提供两条引擎——SQL AI Function 让分析师通过 SQL 直接调用大模型离线推理，MaxFrame 是兼容 Pandas 的分布式 Python 计算框架，支持 CPU/GPU 混合调度。实际案例中，某头部大模型公司用 MaxFrame 实现 PB 级数据预处理，MinHash 算子性能提升 50%+，弹性资源达 16 万核；汽车自动驾驶场景下分布式处理效率提升 40%+。

---

**作者判断**：

作者认为 MaxCompute 正从传统大数据平台演进为 AI 时代数据资产构建的坚实底座。核心观点是 Data 和 AI 不应是割裂的两套系统，而是需要在统一平台上实现多模态数据存储、分布式计算和模型推理的深度融合。通过 SQL 和 Python 双引擎覆盖不同用户群体（SQL 分析师 vs 数据科学家），降低 AI 能力的应用门槛。

---

**核心内容**：

**平台架构（四层）**

- **数据层**：管理结构化与非结构化数据，BLOB 类型支持音频/视频/图片存储，Object Table 打通 OSS/Hologres 外部存储
- **模型层**：托管 XGBoost、LightGBM 传统模型及千问、DeepSeek 等开源大模型，支持调用百炼平台商业模型
- **计算层**：CPU + GPU 混合算力调度，用户声明式指定资源
- **引擎层**：SQL 引擎（SQL AI Function）+ MaxFrame（Python 分布式框架）

**存储管理：多模态数据统一管理**

"湖仓多模态数据统一管理"架构：MaxCompute 内表（BLOB 类型）和外部存储（OSS）统一存放图片/视频/音频，通过 Max Meta 统一元数据服务 + Storage API 实现跨存储引擎的数据访问，无需移动数据。支持单表多模态一行多列混存（BLOB + JSON），将音视频图文数据与元数据、Prompt 统一存储在同一表中。

**计算引擎：SQL AI Function**

SQL 分析师可通过 SQL 函数直接调用大模型对结构化数据进行离线推理，极低门槛。

**计算引擎：MaxFrame**

MaxFrame 是兼容 Pandas DataFrame 标准的分布式 Python 计算框架，三大核心优势：
- **异构算力混合调度**：同一作业中混合 CPU + GPU，编程接口灵活指定
- **分布式数据处理算子**：兼容 Pandas、XGBoost、LightGBM，作业自动分布式执行
- **稳定便捷开发体验**：与 DataWorks 深度集成，支持自定义镜像、OSS 挂载、AI 助手

MaxFrame 内置 AI Function，集成 Qwen3（8B/14B/32B）和 DeepSeek-R1-Distill-Qwen 等大模型，SDK 调用即可指定模型、设置 Prompt、传入参数进行大规模离线推理。

开发环境支持：VS Code、Jupyter Notebook（pip install maxframe）、DataWorks Notebook、DataWorks 数据开发、MaxCompute Notebook。

**场景一：大模型数据预处理**

某头部大模型公司面临 PB 级数据存储、10 万核+弹性资源、数据安全与权限管理等需求。基于 MaxFrame 实现完整数据预处理 Pipeline：
- MinHash 算子性能提升 50%+
- 单次任务稳定运行 300 万核时
- 弹性资源达 16 万核（超出要求的 10 万核）
- PB 级数据处理周期大幅缩短

技术架构：DataWorks Notebook 开发 → Pipeline 编排 → MaxCompute 弹性调度 → 数据存储至内表 → DataWorks 数据地图查询分发。

**场景二：汽车具身智能数据预处理**

车端持续产生海量多模态数据（图片/音视频/雷达/GPS），以 ROS bag 文件存储。基于 MaxFrame 实现端到端处理：
- 弹性计算资源应对波峰波谷
- 分布式处理效率相比单机提升 40%+

**场景三：多模态数据处理（车联网）**

某客户需对采集的图片/视频进行模式识别与打标。MaxCompute 通过 Object Table 统一管理多模态数据，MaxFrame 内置 MinHash 等算子并支持自定义镜像（如 yolo11n）管理模型与依赖，用户灵活设置并发度提升效率。

**场景四：AI Function 图片打标**

结合 AI Function 实现自动化图片打标和向量化，调用大模型进行图像 Embedding 以支撑后续检索分析。
