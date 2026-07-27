---
title: Apache Polaris
description: Apache Polaris 开放目录——基于 Iceberg REST Catalog 协议，Snowflake 开源并由 Snowflake 和 Databricks 联手推进，2026 年 7 月从 Apache 孵化器毕业
aliases: [Polaris, Apache Polaris, Polaris Catalog, Iceberg REST Catalog, 开放目录]
tags: [big-data, tool, concept]
sources: [2026/07/26/从 Arrow 到 Iceberg 到 Polaris 到 Ossie：语义标准化的最后一块拼图.html]
created: 2026-07-27
updated: 2026-07-27
---

# Apache Polaris

Apache Polaris 是 [[iceberg|Apache Iceberg]] 生态的**开放目录（Catalog）**项目，基于 Iceberg REST Catalog 协议实现。2024 年由 Snowflake 开源，Snowflake 和 Databricks 联手推进 Apache 孵化，2026 年 7 月从 Apache 孵化器毕业成为顶级项目。

在 [[open-data-stack-evolution|五层开放数据标准栈]]中，Polaris 占据**目录 & 治理层**。

## 解决的问题

Iceberg 统一了"表"的定义，但留下一个问题：**表多了，怎么找？**

实际场景：Iceberg 表在 Snowflake 里有一套、Databricks 里有一套、Dremio 里还有一套，同一个"客户表"三份，同步靠 Airflow 定时任务。权限更麻烦——谁能在哪个表上做什么，得在三套系统里各配一遍。

Polaris 解决的核心问题：**"你的 Iceberg 表我可以直接读，不用先同步到我的系统里"**。目录是共享的，权限跟着目录走，每套工具不用各搞一套。

## Iceberg REST Catalog 协议

Polaris 的前置条件是一个共识——Iceberg 社区先推出了 **Iceberg REST Catalog 协议**：不要求统一平台，只要求各方同意一个查询目录的接口标准。这是 Polaris 能被跨厂商接受的技术基础。

## 为什么能成：竞品共识

Polaris 之所以能成，**不是因为它技术多牛——而是 Snowflake 和 Databricks 这对竞争对手同时认了它**。两个最主要的 Iceberg 生产用户都同意用一个标准做目录，这标志着开放标准博弈中最难的一步（"竞品也同意用"）被跨过。

## 在五层栈中的位置

```
语义层         Ossie     ← 字段的业务含义
目录 & 治理    Polaris   ← 数据在哪？谁能看？
表格式层       Iceberg   ← 表怎么管？怎么改？
内存交换层     Arrow     ← 数据怎么传？格式是啥？
文件格式层     Parquet   ← 数据存成什么？怎么读？
```

Polaris 是承上启下的关键层：对下依赖 Iceberg 统一表格式（否则不知道管的是表还是文件、物化视图还是快照），对上为 [[apache-ossie|Ossie]] 语义模型提供挂载点（字段定义该跟表走还是跟目录走——Polaris 回答了这个问题）。

## 迁移建议

如果企业内部已有 Snowflake Horizon 或 Databricks Unity Catalog，可以不急着换 Polaris——但**应该开始把权限模型往 Iceberg REST 协议的方向靠**，因为迁移成本会越来越低。

## 相关页面

- [[open-data-stack-evolution]] — 五层开放数据标准栈完整演进
- [[iceberg]] — 上游依赖，表格式层
- [[apache-ossie]] — 下游依赖，语义层
- [[gravitino]] — Apache Gravitino 统一元数据湖（也提供 REST Catalog）
- [[agentic-data-cloud]] — Google 跨云 Lakehouse 使用 Iceberg REST Catalog 联邦读取 Polaris
