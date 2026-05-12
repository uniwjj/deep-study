---
title: EMR Skills
description: 阿里云 EMR AI Native 平台——数据处理/实时分析/系统运维三大 Skill 类、Spark Skills 驱动智驾图片标注、多模态 AI Function、全栈 Serverless 免运维
aliases: [EMR Skills, EMR AI Native, EMR Agent]
tags: [big-data, ai-agent, platform]
sources: [2026/05/12/06. 基于 EMR Skills 实现智驾场景的自动化数据标注.pdf, 2026/05/12/DataWorks DataAgent分享录音.md]
created: 2026-05-12
updated: 2026-05-12
---

# EMR Skills

EMR 定位为**面向 Agent 的数据与计算能力平台**，不再只是“集群产品”，而是对外提供三类核心能力。演讲人：董亚军（EMR 产品专家）。

## EMR AI Native 全景图

### 四层架构

```
应用场景层：智能数据分析 / 智能数据开发 / 智能运维管理 / 多模态数据处理
    ↓
AI 能力层：EMR AI 助手 / 智能开发助手 / AI Function / Agent Skills / 多模态处理引擎
    ↓
AI 基础设施层：百炼 Qwen 系列 / Token 管控 / 安全合规 / 推理优化
    ↓
计算引擎层：EMR on ECS/ACK / Serverless StarRocks / Serverless Spark
    ↓
数据底座层：OSS/NAS/CPFS / DLF Catalog / Paimon/Iceberg/Fluss
```

### AI 能力层详情

| 能力 | 功能 |
|------|------|
| **EMR AI 助手** | 实例管理、任务诊断优化、错误根因分析、健康巡检、成本优化 |
| **智能开发助手** | SQL 智能生成、SQL 方言迁移、代码生成、执行计划诊断 |
| **AI Function 体系** | 情感分析、文本翻译、实体识别、图像分析、文档理解、音频处理、时序预测、内容安全 |
| **Agent Skills** | 集群/实例管理、作业管理、弹性管理、资源管理、运维管理、SQL 执行、元数据查询、智能诊断 |
| **多模态处理引擎** | Embedding、批量模型推理、非结构化数据加工、全文检索、向量检索、混合检索 |

## EMR Skill 能力全景

三大子产品通过 MCP Skill 接入 AI 助手，覆盖集群管理、计算作业、分析数据库全链路运维。

### EMR on ECS 集群管理 Skill

`alibabacloud-emr-cluster-manage`

管理 5 种集群类型（HA/普通部署、存算分离/一体架构）的全生命周期：

| 功能 | 能力 |
|------|------|
| 生命周期管理 | 创建、状态查询、克隆、删除 |
| 节点组管理 | 5 种节点角色（MASTER/CORE/TASK 等）、新建节点组、手动扩缩容 |
| 弹性伸缩 | 定时策略与指标策略（CPU/内存/YARN 负载触发）、伸缩活动历史 |
| 运维诊断 | 集群与节点巡检、续费管理、创建失败诊断 |

### EMR Serverless Spark 工作空间管理 Skill

`alibabacloud-emr-spark-manage`

| 功能 | 能力 |
|------|------|
| 工作空间管理 | 创建、查询、删除工作空间，查询可用引擎版本 |
| 作业管理 | 提交 Spark 作业（JAR/PySpark/SQL）、查询状态与日志、取消作业 |
| Kyuubi 交互查询 | 创建/启停 SQL 网关、JDBC 连接、Token 管理、取消查询 |
| 资源队列管理 | CU 扩缩容、查询队列状态 |
| 会话集群 | 创建/启停/删除会话集群，支持交互式开发 |

### EMR Serverless StarRocks 管理 Skill

`alibabacloud-emr-starrocks-manage`

| 功能 | 能力 |
|------|------|
| 实例管理 | 创建（存算一体/分离）、查询状态、释放、恢复暂停 |
| 弹性扩缩容 | 节点/磁盘扩缩容、计算组节点增减，均支持预检 |
| 配置管理 | 配置修改（预检+回滚）、版本升级、SSL、付费转换 |
| 运维管理 | 重启实例/计算组/节点、操作历史查询 |
| 网络管理 | 网关增删改查、隔离、公网 SLB 开关 |

## Agent 驱动智驾图片打标

### 架构

```
控制编排层           计算引擎层           数据源层              模型服务层
DW DataAgent  →     Serverless Spark  →  DLF & OSS    →    Qwen3.6-plus
  Skills Library                          camera_front/
  · Spark Skills                          ├─ img_01.jpg
  · Kyuubi Skills                         ├─ img_02.jpg
                                          └─ ...
```

### 执行流程

1. **Action**：Agent 理解用户意图 → 生成 SQL 代码 → 提交任务
2. **Spark 执行**：解析 Prompt → `read_files` 读取 OSS 图片 → `ai_query()` 调用 Qwen3.6-plus（可指定模型如 `model='qwen3.6-plus'`）
3. **输出**：结构化标注结果 `["警车", "吊车"]`

使用 Serverless Spark 内置的 `read_file` 表函数直接读取 OSS 文件：

```sql
SELECT ai_query(prompt, read_file(path)) FROM read_file('oss://bucket/path/*.jpg');
```

整个流程简化为标准 ETL 流水线：读取 OSS 文件 → SQL 定义 Prompt → 调用 AI 函数 → 输出结构化结果。Spark 侧提供 `model` 参数切换模型（如 Qwen 3.5/3.6/embedding），支持 JPG/PNG 后缀过滤。

### 核心优势

- **Agent 原生集成**：一键添加 Skill，开箱即用
- **核心场景覆盖**：工作空间、作业、查询、资源四大管理场景
- **安全合规**：基于 RAM 权限体系，细粒度访问控制
- **Serverless 免运维**：无需管理基础设施，按需弹性扩缩容
- **生产级稳定性**：企业级可靠性保障，完善的容错与重试

## EMR AI Native 展望

- EMR Skills 支持 Spark 诊断与优化
- Serverless Spark **Daft 集群 GA**，支持 Auto Scaling、GPU 调度和推理优化
- Serverless StarRocks 基于 Lakehouse 向量检索 & 全文检索 GA
- Ray 集群支持 Auto Scaling、GPU 调度和推理优化
- 智能 AI 助手接入飞书等通信工具
- EMR Skills 支持参数优化、索引/分区优化
- Spark 多模态算子市场 GA

## 相关页面

- [[dataworks-data-agent]] — DataWorks Data Agent 产品
- [[maxcompute-skills]] — MaxCompute 同类 Skills 体系
- [[hologres-skills]] — Hologres Skills
- [[flink-skills]] — Flink Skills（Flink + EMR 联动）
- [[dataworks-stock-case]] — Agent 驱动数据处理的实践案例
- [[agent-mcp-protocol]] — MCP 协议原理
