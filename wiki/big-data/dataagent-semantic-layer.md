---
title: DataAgent 语义层
description: DataWorks DataAgent 语义层的具体内容——从分享录音和数据开发 SKILL 中提取的结构化领域知识，包括节点类型体系、连接器参数、FlowSpec 规范、调度与依赖语义、运行时约束等
aliases: [语义层, semantic layer, 数据语义层]
tags: [big-data, ai-agent, meta]
sources: [2026/05/12/DataWorks DataAgent分享录音.md, 2026/05/14/DataAgent语义层分析.md]
created: 2026-05-14
updated: 2026-05-14
---

# DataAgent 语义层

## 概述

DataAgent 分享中提到"语义层是共享的"，但描述含糊。经对照数据开发 SKILL（`alibabacloud-dataworks-datastudio-develop`）和分享录音，语义层在实物层面指的是以下结构化领域知识。

## 分享中的描述

### 录音（许志）

- 两个 Agent 共享同一份数据语义与上下文，避免上下文割裂
- 上下文可持久化——Claw 能读取 Code Agent 开发过的内容
- 内核是可被两种模式共享和使用的"受控执行内核"
- Agent "懂数据、懂代码、懂安全权限体系"

### PDF 幻灯片

> 数据语义层：统一的数据字典、术语映射和业务上下文

## SKILL 中的实际内容

### 节点类型体系

约 150+ 种节点类型，每种包含：`command`（节点类型标识）、`contentFormat`（sql/shell/python/json/empty）、文件扩展名、`datasourceType`（odps/hologres/flink/emr/mysql/clickhouse/starrocks）、运行时约束。

常用映射：

| 用途 | command | 扩展名 | datasourceType |
|------|---------|--------|----------------|
| MaxCompute SQL | ODPS_SQL | .sql | odps |
| Hologres SQL | HOLOGRES_SQL | .sql | hologres |
| Flink 流式 SQL | FLINK_SQL_STREAM | .json | flink |
| EMR Hive | EMR_HIVE | .sql | emr |
| Serverless Spark | SERVERLESS_SPARK_SQL | .sql | emr |
| 数据同步 | DI | .json | — |
| Shell | DIDE_SHELL | .sh | — |
| Python | PYTHON | .py | — |
| 虚拟节点 | VIRTUAL | .vi | — |

### 数据连接器

48 个数据源，每个定义了读写参数、字段类型映射、写入模式、通道配置：

- 写入模式：MySQL insert/update/replace，Hologres replace/ignore/update，MaxCompute truncate/非truncate，OSS truncate/append/nonConflict，ES index/update，MongoDB isReplace，Redis string/list/set/zset/hash
- 通道配置：并发度 1-32，脏数据策略 Ignore/Tolerance/ZeroTolerance，executeMode distribute/null
- DIJob 结构：type + version + steps + order(hops) + setting + extend

### FlowSpec 规范

- `version: "2.0.0"`，13 种 `kind` 枚举
- 必填字段：name、id（与 name 一致）、script.path、script.runtime.command
- `outputs.nodeOutputs[].data`：格式 `${projectIdentifier}.NodeName`，项目内全局唯一
- `dependencies[].nodeId`：自引用，depends[].output 为上游输出
- datasource、runtimeResource、trigger、rerunMode 等配置字段

### 调度语义

- Cron 6 位表达式 `ss mm HH dd MM ?`
- 实例生成 T+1 或 Immediately
- 业务日期 = 运行日期 - 1
- 空跑实例：非调度日生成的虚拟成功实例，用于衔接依赖
- 批量实例生成窗口期 23:30-24:00

### 依赖语义

- Normal：同周期标准依赖
- CrossCycleDependsOnSelf：依赖自己上一周期
- CrossCycleDependsOnOther：依赖其他节点上一周期
- 根节点：`${projectIdentifier}_root`
- nodeId 为自引用，depends[].output 为上游输出

### 占位符机制

- `${spec.xxx}` / `${script.xxx}`：从 dataworks.properties 替换
- `${projectIdentifier}`：项目标识
- 调度参数：`$yyyymmdd`、`$[yyyymmdd]`、`$[yyyymmdd-1]`、`${bizdate}`、`$gmtdate`

### 部署状态机

- create-pipeline-run → get-pipeline-run → exec-pipeline-run-stage
- Pipeline 终态仅 Success 表示成功部署
- Stage：Build（自动）/ Check / Deploy / Offline（手动推进）

### 工作流控制节点

- CONTROLLER_BRANCH：条件分支
- CONTROLLER_JOIN：分支归并
- CONTROLLER_CYCLE：do-while 循环
- CONTROLLER_TRAVERSE：for-each 遍历
- CONTROLLER_ASSIGNMENT：赋值传递

### 运行时约束

- ODPS_SQL：128KB / 200条SQL / 10000行 / 10MB
- PYODPS3：50MB / 16CU，仅 print() 输出
- DI 并发 1-32，分布式 >= 8
- 超时默认 4 HOURS，重试间隔 180000ms

### 示例中的隐性惯例

SKILL 示例 SQL 中使用但未显式定义的写法：
- 表前缀 `ods_`、`dwd_`、`ads_`、`dim_`
- `INSERT OVERWRITE TABLE ... PARTITION (dt='${bizdate}')`
- `COALESCE` 空值处理
- `CURRENT_TIMESTAMP AS etl_time` 审计字段
- `LEFT JOIN dim_*` 维度关联

## 结论

分享中语义层描述含糊，实物层面的语义层是：
1. **数据字典**：节点类型→数据源类型→代码格式→扩展名的完整映射
2. **术语映射**：48 个连接器的读写参数、写入模式、字段类型
3. **业务上下文**：调度语义（T+1、空跑、Cron）、依赖语义（Normal/CrossCycle/根节点）、部署流程
4. **运行时上下文**：代码大小、超时、重试、并发度等约束
5. **隐性惯例**：数仓分层前缀、分区覆写模式等在示例中暗示的规范

## 相关页面

- [[dataworks-data-agent]] — DataWorks Data Agent 产品
- [[maxcompute-skills]] — MaxCompute MCMCP 与 Skills
- [[hologres-skills]] — Hologres CLI 与 Skills
- [[flink-skills]] — Flink Agent Skills
- [[emr-skills]] — EMR Skills
- [[openai-data-agent]] — OpenAI 内部 Data Agent（对照语义正确性问题）
