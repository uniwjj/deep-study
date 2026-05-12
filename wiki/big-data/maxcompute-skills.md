---
title: MaxCompute Skills
description: MaxCompute 通过 MCMCP 服务 + Information Schema 语义包 + MaxFrame Coding Skill 构建的 Agent 可调用能力层——覆盖数仓运维、多模态开发、数据分析全场景
aliases: [MaxCompute Skills, MCMCP, MC Skills]
tags: [big-data, ai-agent, platform]
sources: [2026/05/12/03. 基于 MaxCompute Skills 简化数仓运维加速多模态开发.pdf, 2026/05/12/DataWorks DataAgent分享录音.md]
created: 2026-05-12
updated: 2026-05-12
---

# MaxCompute Skills

MaxCompute 通过 **MCMCP** 与 **Skills** 结合，将底层元数据、计算和存储服务封装成 Agent 可理解、可执行的统一能力接入层，让企业客户通过 Agent 完成超大规模数据分析、高性价比多模数据加工和智能化运维。演讲人：孙戴博/六稻（MaxCompute PD）。

## 接入架构

```
用户 Agent 生态（OpenClaw / DataWorks Agent / Qwen Code / QoderWork）
              ↓
        MC Skills 集合（语义包 + 常用命令 + 开发模板 + 使用限制）
              ↓
        MCMCP 服务（OpenAPI / StorageAPI / CatalogAPI → 结构化工具）
              ↓
        MaxCompute（Metadata + Compute Engines + Storage）
              ↓
    授权 / 审计 / 数据发现 / 敏感等级 / 脱敏 / 行级权限 / 数据共享
```

## MCMCP 服务

MCMCP 是 MaxCompute 提供的 [[agent-mcp-protocol|Model Context Protocol（MCP）]]服务，接入后用户可通过自然语言对话管理和分析 PB/EB 级数据。

### 方法分组

| 分组 | 方法 | 说明 |
|------|------|------|
| 身份与权限 | `check_access` | AK/SK、STS、Credentials URI 等多认证方式 |
| 元数据 | `list projects/schemas/tables/partitions` | 元数据搜索服务，自然语言找表 |
| Knowledge Catalog | `search_meta_data` | 语义搜索，关联业务术语 |
| 分析计算 | `cost_sql` / `execute_sql` / `get_instance_status` | 执行前成本评估审批，约束仅允许只读查询 |
| 数据管理 | `create_table` / `insert_values` | 建表时提供生命周期、主键、部分列更新等选项 |

## Information Schema 语义包 Skill

系统级运维（Ops）语义包，基于 `INFORMATION_SCHEMA` 元数据视图构建，将底层元数据转化为可自然语言查询的指标和对象。

### 规模

- **54 术语**：口语到治理概念映射
- **34 实体**：表/字段/任务/Quota/Tunnel
- **16 关联**：对象关系可串起来分析
- **35 指标**：存储/成本/资源/安全（如 `CU时 = SUM(cost_cpu / 100 / 3600)`）
- **17 场景**：playbook 直连常见治理问题
- **38 SQL**：已验证查询模板可直接复用

语义包不是 prompt，而是结构化 Knowledge Catalog：
```
information_schema/
├── terms.json
├── entities.json
├── metrics.json
├── joins.json
├── verified-queries.json
└── playbooks.json
```

### 典型运维场景

| 场景 | 典型提问 | 能力 |
|------|---------|------|
| 存储压力 | “哪些表最近占用的存储最高？” | 识别 TOP 表和分区膨胀，定位生命周期异常与冷热失衡 |
| 成本压力 | “为什么这个月成本比上月涨得快？” | 按 owner/project/task_type 拆解 CU 时 |
| 权限审计 | “哪些表的权限暴露面更大？” | 审计 LABEL 保护表与字段级授权 |
| 传输审计 | “最近有哪些公网下载行为？” | 基于 TUNNELS_HISTORY 与 client_ip 回溯 |
| 失败激增 | “最近 7 天失败任务为什么突然变多？” | 结合 TASKS_HISTORY 对比定位异常 |
| Quota 监控 | “哪些 Quota 已经接近资源瓶颈？” | 给出 quota_bottleneck 判断 |

### 更多运维场景

- **作业诊断**：看 LogView → 调优建议，失败原因分析和改写建议
- **Quota 资源管理**：资源状态、资源规划、成本对比分析
- **MMS 数据迁移**：按需执行迁移任务，定时迁移

## 多模态存储与计算

### 存储（Blob 数据类型）

| 特性 | 能力 |
|------|------|
| 单元格容量 | 5GB/cell |
| 事务 | ACID 多模态数据集事务一致性 |
| 托管优化 | Compaction、Tiering、Index Build、Merge、Backup |
| IO 吞吐 | 相对 Object Table 提升 4X |
| 点查过滤 | 数据集标签点查场景性能优于 String/Binary 类型 2X |
| 表类型 | 支持 Append/PK Delta Table |

### 计算（多模态算子）

| 类型 | 能力 |
|------|------|
| 图像/OCR | `IMG` |
| 文档/解析 | `DOC` |
| 音频/ASR | `AUDIO` |
| 视频/切帧 | `VIDEO` |

## MaxFrame Coding Skill

集成到主流 AI 编程助手中，将 MaxFrame 分布式数据处理知识体系注入 Vibe Coding：

### 核心能力

- **Operator Selector**：自动推荐更合适的算子与处理方式
- **API 校验**：实时验证 API 是否真实存在
- **3 阶段开发**：确认需求 → 确认方案 → 生成代码
- **10 模板**：生产级示例覆盖常见开发场景

### 多模态算子举例

| 场景 | 推荐算子 |
|------|---------|
| 逐行文件处理 | `df.apply(axis=1)` |
| 模型批推理 | `df.mf.apply_chunk()` |
| 视频切帧 | `df.mf.flatmap()` |
| 音频合并 | `df.mf.map_reduce()` |
| 并行调优 | `df.mf.rebalance()` |

### 开发模式

访问 OSS 常配合 `@with_fs_mount`，模型加载使用 `_ctx` 缓存减少重复加载，更适合分布式 Worker 批处理。生成代码的最低生产约束包含 session 创建和销毁。

### 演进方向

- 基座：`maxframe-job-coding`
- 专业方向：多模态处理、AI Function、GPU/资源调度

## 多 Agent 接入口

| 入口 | 能力 | 特色 |
|------|------|------|
| **MaxCompute CLI** | `bootstrap` 环境引导、`whoami --json` 结构化输出、`skill install` | JSON 输出 Agent 友好，多认证方式 |
| **数据探索客户端** | 桌面级客户端，本地持久化工作现场 | AI 驱动数据探索，推断 DDL |
| **OpenClaw Sub-Agent** | OpenClaw 做总控，Sub-Agent 做专业执行 | 证据链完整：Plan → Act → Result |

## 相关页面

- [[dataworks-data-agent]] — DataWorks Data Agent 整体产品
- [[maxcompute-data-ai]] — MaxCompute Data+AI 四层架构
- [[hologres-skills]] — Hologres 同类 Skills 体系
- [[flink-skills]] — Flink Agent Skills
- [[emr-skills]] — EMR Skills
- [[agent-mcp-protocol]] — MCP 协议原理
