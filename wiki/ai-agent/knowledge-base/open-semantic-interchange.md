---
title: Open Semantic Interchange（OSI）——语义层的开放标准与 Bottom-up 指标治理
description: 赵恒（datus.ai）在 2026 Agent 大会的演讲——语义层正在成为 Agent 的"契约语言"却在重走 BI 孤岛老路。OSI 开放标准（2025.9 由 Snowflake 联合发起）把 N×N 语义锁定降为 N，让指标 define once, query anywhere；语义层之上长出 Ontology 的两种路线（Knowledge Graph vs Flattened GraphRAG）；Datus OSI Engine；Bottom-up 指标体系方法论（从历史 SQL/Dashboard 长出指标，像管代码一样治理）
aliases: [OSI, Open Semantic Interchange, 开放语义交换, Datus, 指标开放标准, 语义层契约语言]
tags: [ai-agent, big-data, concept]
sources: [2026/08/11/面向Agent的知识工程与本体语义构建论坛/02-赵恒-Open Semantic Interchange：为什么本体需要一个开放标准的指标.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# Open Semantic Interchange（OSI）

> 核心论点：**语义层，正在成为 Agent 的"契约语言"，但今天它却在重走 BI 时代的孤岛老路**。契约需要标准——语义层若人人一套，Agent 就永远拿不到可信、可移植的业务口径。

演讲来源：2026 Agent 大会「面向Agent的知识工程与本体语义构建论坛」，赵恒 @datus.ai。

## 孤岛现状：语义层的各家各话

- Snowflake / Databricks 各自的 Semantic View / Metric View
- Looker · Tableau · Superset 等 BI 厂商纷纷发布语义层
- Cube · dbt MetricFlow 等第三方独立方案
- 差异带来新的语义孤岛与生态锁定

**真实案例**：一个 Snowflake 客户却有四套指标口径——①Tableau Semantics（沉淀已久的可视化语义）、②LookML（Looker 里的指标模型）、③StatSig metrics（实验平台里的指标）、④Snowflake Semantic View（数仓原生语义对象）。数据层已统一，但"数仓统一 ≠ 指标统一"，Agent 不知道该信哪个"收入"。

## 行业动向：两家龙头都在语义层上叠 Ontology

| | Snowflake | Databricks |
|--|-----------|------------|
| 底座 | Cortex Agent + Semantic View | Genie + Metric View |
| 本体 | Ontology-grounded Cortex Agents；KG_NODE / KG_EDGE + Recursive CTE；语义层之上叠本体推理 | Genie Ontology（2026.6 发布）；OntoRank 判定定义权威度 |
| 效果 | — | 首答正确率 84.5% vs 通用 52.4%（详见 [[genie-ontology]]） |

殊途同归：**先有治理良好的 Semantic Layer，再在其上生长出 Ontology 供 Agent 推理**。

## 两个问题

- **Q1：如何构建统一的语义层？** 在多引擎、多 BI、多方案并存的现实里，怎样得到一套可移植、可治理的指标口径（→ OSI）
- **Q2：从语义层长出的 Ontology 是什么形态？** 指标之上的本体如何组织、如何被 Agent 消费、以什么产品形态落地

## 问题一：统一语义层

### Semantic Layer / Model / View 三形态

| 形态 | 定义 |
|------|------|
| Semantic Layer | 物理表之上的治理层：把 measures / dimensions / relationships / time 声明式定义一次，成为单一事实源——让 Agent 面对业务概念，而非裸 schema |
| Semantic Model | 语义层的"代码化"定义：以 YAML 描述实体、join、指标，随项目版本管理（dbt / MetricFlow）——偏工程：可 review、可 diff、可复用 |
| Semantic View | 语义层的"仓内对象"形态：以 DDL 建在数据库里的一等对象，被 Cortex Analyst / Agent 直接消费——偏消费：治理与权限随仓库继承 |

定义一次，处处一致——让指标与业务逻辑，和具体 dashboard 解耦。

### 同质不同名：抽象一致，落地各异

各方案都在声明 entities / dimensions / measures / metrics，但落地各不相同：

| 对比维度 | 差异 |
|---------|------|
| 存储位置 | 仓内对象 vs git 代码文件 |
| 引擎绑定 | 绑定单一平台 vs 引擎独立 |
| 表达式方言 | 各家 SQL 方言不通用 |
| join / 关系语义 | RELATIONSHIPS · cardinality · entities |
| 权限模型 | RBAC 继承 vs 语义层自带规则 |

（示例：dbt MetricFlow semantic model YAML、Snowflake semantic model YAML（含 synonyms）→ 生成/转为 Semantic View；Snowflake `CREATE SEMANTIC VIEW sales_sv` DDL（TABLES/RELATIONSHIPS/FACTS/DIMENSIONS/METRICS）；Databricks Metric View（Unity Catalog，version 1.1，用 fields / measures）——Databricks 叫 Metric View，与 Snowflake Semantic View 同质不同名。）

共性足以标准化，差异足以造成锁定——中间缺的，是一个**厂商中立的"最大公约数"**。

### 答案：OSI（Open Semantic Interchange）

- **2025.9 由 Snowflake 联合 Salesforce/Tableau、dbt Labs 等发起**
- **v1.0 规范 2026.1 finalized，Apache 2.0 许可，拟捐 Apache 基金会**
- **60+ 成员**：Databricks、Cube、AtScale、dbt、BI 与 catalog 厂商
- 核心类：Model / Datasets / Fields / Metrics / Dimensions / Relationships
- 多方言表达式（dialects，如 ANSI_SQL）——同一指标语义可移植到多引擎
- Converters：OSI ↔ Snowflake、OSI ↔ dbt

**OSI 不取代任何一家，而是做它们之间的"互换格式"——把 N×N 的锁定，降为 N。**

### 指标即接口：统一指标能做什么

Metrics = API = SQL = LLM tools = YAML。技术上：上卷 Roll-up / 下钻 Drill-down / 归因 Attribution / 预测 Forecast / 物化加速 Materialization。业务上：一套北极星指标 / AARRR 体系引导业务，数仓开发 · BI 报表 · AI 问数 · AB 实验 · 人群圈选全部说同一套指标语言。

### 表达能力的边界

- OSI 预留 **custom extension** 供各引擎表达规范外的语义
- 跨表粒度对齐 & Join（多表 fan-out、join 语义）规范未强制统一
- 复杂窗口、留存 / 漏斗等衍生指标表达能力各家不同
- **OSI 基础规范 ≈ 打通 80% 基础原子指标互通；衍生指标 · 跨表语义 · 加速方案取决于引擎实现**——衍生表达与加速，正是引擎的竞争区

### Datus OSI Engine

原生 Rust 的 OSI 解析 & 执行引擎：

- **parser（YAML → SQL）达 MetricFlow 的 10-20x 性能**
- Datus 扩展：Fan-out 正确性、时间表达式语义准确性
- 跨数据源结果一致：DuckDB / StarRocks / ClickHouse / Doris / TiDB / Trino / Postgres / MySQL / Snowflake / BigQuery / Databricks
- 编译管线：OSI YAML → `osi-model` → Semantic IR → `osi-compiler` → Logical Plan → `osi-planner` → SQL AST（polyglot-sql）→ Dialect SQL

## 问题二：从语义层长出的 Ontology

### 什么是本体（Ontology）——两种形态

| | ① 语义网时代 | ② Palantir 之后 |
|--|-------------|-----------------|
| 技术 | RDFS / OWL | datasets / models → Object Types · Properties · Links |
| 内容 | classes / properties / relationships 形式化"描述世界" | 再加入 Actions · Functions · 安全机制 |
| 定位 | 一张静态的"世界知识图"（知识表示与逻辑推理） | 组织的 digital twin / operational layer：从"描述世界"→"驱动业务操作" |

本场要的正是第二种：**可被 Agent 消费、能驱动操作的本体——长在语义层与指标之上**。

### 本体形态：用表建图，无需图数据库

以 Snowflake Ontology 为例：语义层给出实体 / 指标 / join，本体再叠层级、同义、领域关系：

- **KG_NODE**：实体（id / type / 属性 VARIANT）
- **KG_EDGE**：关系（src / dst / type / 元数据）
- **Recursive CTE 动态多跳遍历层级**（subClassOf 可变长展开，数据驱动）

"本体不必外挂图数据库：节点边两张表 + 递归 CTE，就能在仓内做层级 / 传递 / 同义推理。"

### 两条 Agent 消费路线

**路线一：Knowledge Graph + Semantic Model（直接出 SQL · 更探索）**

- Cortex Agent 编排 **7 个工具**（4 存储过程 + 2 语义视图 + 基线）
- 图遍历工具：get_ancestors / expand_cohort / get_hierarchy_path
- 运行时遍历图动态展开层级：确定性强、覆盖深（如 693 个后代跨 10+ 层）
- 需精确概念名、无同义解析——灵活但方差大，最考验 Agent 的工具选择

**路线二：Flattened GraphRAG + Semantic View（直接用指标 · 更稳定）**

- 建索引：本体表 → Recursive CTE 聚合后代属性 → 概念画像（名/定义/同义/邻居）→ Cortex Search 关键词+向量索引
- 查询仅 **2 个工具**：Cortex Search 检索概念画像 → 提取类别 → Cortex Analyst SQL → 答案
- 同义解析强、方差最低

**Benchmark（Snowflake 生物医学 22 题，首答成功率）**：

| 方案 | 首答成功率 |
|------|-----------|
| Semantic View 基线（1 工具） | 50.0% |
| + Knowledge Graph（7 工具） | 60.0% |
| + GraphRAG（2 工具） | 70.0% |
| + GraphRAG & 术语映射 | **78.2%（最优）** |

### 示例：Snowflake 足球知识图谱（ontology-on-snowflake）

- 场景：足球领域，15 俱乐部 / 41 球员 / 11 教练 / 20 比赛；元数据驱动，对标 Palantir
- **具体层 Concrete KG**（KG_NODE + KG_EDGE）：PLAYER、COACH、CLUB、MATCH；PLAYS_FOR、PLAYED_IN、HOME/AWAY、COACHES
- **抽象层 Ontology**（VW_ONT_*）：Thing / Person / Org / Event / Player+Coach / Club / Match；`WORKS_FOR = PLAYS_FOR ∪ COACHES`、`PARTICIPATES_IN = PLAYED_IN ∪ HOME/AWAY_TEAM`
- 同一份点边 → 抽象归并，跨类型推理：具体层答"谁效力皇马"，抽象层答"皇马所有相关人员（球员+教练）"

**给 Agent 配置的 8 个工具（3+3+2）**：

- A 类 · Cortex Analyst text-to-SQL（绑定 Semantic Model YAML）：query_soccer_data、query_soccer_ontology、query_metadata_governance
- B 类 · 图分析（generic → SPCS Service Function · NetworkX）：centrality_tool（中心性）、community_detection_tool（Louvain）、shortest_path_tool（最短路径）
- C 类 · pandas 程序补充工具：temporal_analysis_tool（时序演化）、transfer_analysis_tool（转会网络）

### OSI 0.2.0：Ontology Preview

**Apache Ossie · v0.2.0.dev0（2026-05）**：在指标之上，声明本体：

- **Concepts**：EntityType（实体）/ ValueType（值类型）；内建 Any · Integer · String · Date…；extends 继承
- **Relationships**：roles + multiplicity（ManyToOne / OneToOne）；**verbalizes** 自然语言模板供 LLM 消费（如 `'{Person} earns {Salary}'`）
- **Business rules**：derived_by（派生/递归，如 ancestor_of）+ requires（约束）
- **Ontology mappings**：逻辑字段 → 概念/关系（object · link · referent mappings）

> 指标统一"怎么算"，本体统一"是什么、怎么连、怎么说"——verbalizes 让 Agent 拿到可验证语义。

（Ossie 在开放数据标准栈中的位置见 [[apache-ossie]] 与 [[open-data-stack-evolution]]。）

### 价值证明：从数据到行动

数据开发闭环（数据开发 → 数据质量 → 指标开发）→ 指标消费闭环（指标问数 → 生成 Dashboard/Report → 指标归因）→ **本体洞察层 Insight & Action**：不止看数/问数，而是从更高层给出数据驱动的行动建议与洞察。

## Bottom-up 指标体系方法论

### 冷启动：如何初始化指标体系

两种视角对齐（业务导向：要回答什么业务问题 → 需要哪些 Semantic View/Metrics；工程导向：数据怎么连 → 需要哪些 Semantic Model / key filter & relation），三条来源汇入：

1. **从历史 SQL 抽取**：挖掘聚合模式 → 候选指标，带置信度与去重
2. **从历史 Dashboard 抽取**：从既有 BI 报表反解出指标定义
3. **从开发中新建**：在数据建设流程里顺带沉淀指标

> 指标不是凭空规划出来的，而是从历史 SQL、Dashboard 与开发产物里"抽"与"长"出来的。

### Top-down 为什么失败

- **政治**：谁说了算？口径归属、优先级、责任边界谈不拢——统一还没产生价值，就先陷入拉锯
- **技术**：生产端与消费端打不通——定义在一处、查询在另一处，指标目录与真实 SQL/Job 各行其是
- 结局：一个漂亮但没人用的指标目录——不是不需要统一，而是想在产生价值前先统一

### Data Governance for Agent：像管代码一样管 SQL / Job / Metrics

把指标纳入**版本 / PR / Review / Merge / 血缘**管理。指标是被反复查询、验证的工程产物，天然适合用软件工程的方式治理——Agent 消费的每一个口径都可追溯、可回滚、可审计。治理的对象不再是静态目录，而是活的工程产物。

### 产品形态：Datus Semantic Hub

像 GitHub 一样 push / pull + PR review 逐级晋升：个人 Workspace（MetricFlow 级个人指标体系）→ PR · Review · Merge（评审合并）→ 公司 Semantic Hub（org 级指标树、企业唯一真相来源；监控 · 血缘 · 权限）。指标生命周期：**Unverified → Verified → Certified → Deprecated → Archived**。

容许不一致：先在 project / workspace 交付价值，再经 PR 逐步收敛，晋升为公司级指标资产。

### Bottom-up 路径：先交付，再统一

Project 级先交付 → 被反复查询验证 → PR 收敛统一 → Enterprise 指标资产。

> 企业级 ontology 不是手工画出来的，而是从被反复查询、验证的 metrics 中"长"出来的。**Metric 是 data ontology 的基石**——先让指标在真实使用中被检验，再渐进统一到 enterprise。

## 三个 Takeaway

1. **语义层是 Agent 的契约语言**：准确、可治理的指标 / 知识库 / 本体，比更大的模型更能让 Agent 稳定回答业务问题
2. **开放胜过孤岛：OSI**：厂商中立的最大公约数 spec，把 N×N 的语义锁定降为 N，让指标 define once、query anywhere
3. **Bottom-up 胜过 Top-down**：像管代码一样管指标，从真实产物里长出本体，先交付价值再渐进统一

（附：Datus 开源项目 github.com/Datus-ai/Datus-agent，提供 Star 与 POC 报名。）

## 相关页面

- [[apache-ossie]] — Apache Ossie 语义层项目（开放数据标准栈视角），OSI 0.2.0 Ontology Preview 的承载项目
- [[open-data-stack-evolution]] — 开放数据标准栈十年演进（Parquet→Arrow→Iceberg→Polaris→Ossie）
- [[genie-ontology]] — Databricks Genie Ontology（OntoRank、84.5% vs 52.4%），本演讲的 Databricks 对照
- [[ontological-semantic-layer]] — 本体化语义层，语义层与本体关系的概念框架
- [[data-agent-semantic-stack]] — Data Agent 8 层语义栈，本体语义层（OWL/SKOS + 图数据库 + SPARQL/MCP）与 OSI 本体的对照
- [[graphrag]] — Flattened GraphRAG 路线的基础范式
- [[dataagent-semantic-layer]] — DataWorks DataAgent 语义层实物层面
- [[ontology]] — 本体论五要素框架
- [[data-agent-vs-ontology-agent]] — Data Agent 与 Ontology Agent 推理范式对比
- [[xiaomi-dimi-data-agent]] — 小米 Data Agent 语义层 Harness 实践（同论坛演讲）
- [[databricks-genie]] — Databricks Genie 企业级 Data Agent
