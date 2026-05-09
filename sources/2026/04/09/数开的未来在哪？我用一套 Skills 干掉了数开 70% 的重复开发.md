---
ingested: 2026-05-09
wiki_pages: [big-data/data-dev-skills-automation]
---

## 数开的未来在哪？我用一套 Skills 干掉了数开 70% 的重复开发

**文章信息**：
- **原文标题**：数开的未来在哪？我用一套 Skills 干掉了数开 70% 的重复开发
- **来源**：https://mp.weixin.qq.com/s/sjlVgkZ6PlP1cPLlCRvmkw
- **作者**：小友数研
- **发表时间**：2026-04-09 08:35

---

**一段话总结**：

作者用 Claude Code Skills 封装了一套数仓开发工具链，将查表、写验证 SQL、部署节点等重复操作自动化，数据开发中真正写 SQL 的时间从不到 30% 大幅提升。核心 Skill 涵盖四大场景：开发（查表结构、跨库核对、本地 SQL 直跑 MaxCompute）、测试（一行命令跑完行数/唯一性/非空率/跨层一致性/环比波动）、部署（批量提交+部署节点、创建 DI 同步任务）、运维（全链路状态查看、日志排查+重跑）。真实案例中，一条 ADS 表停更 7 天的故障用对话窗口 3 秒定位根因，三小时完成整条链路重建。底层对接 MaxCompute 和 DataWorks API，用户只需定义"要什么"。

---

**作者判断**：

作者的核心立场：数据开发的价值在于理解业务、设计模型、保障质量，而不是在控制台上点来点去。用 Skills 自动化重复操作后，开发者有更多时间思考表粒度对不对、链路能否撑住未来需求。工具的进化方向是让人回归思考本身。作者建议从最常用的操作开始构建 Skill（他的第一个是 `/dw-status`），不需要一步到位。

---

**核心内容**：

**问题背景：数据开发的时间分配失衡**

数据开发最耗时的不是写 SQL，而是围绕 SQL 的一切：

- 查表结构要开控制台或写 DESC，一次排查开 5 个 Tab
- ETL 写完还要到控制台建表、建节点、配依赖，全是表单弹窗
- 部署一个节点 8 步点击（保存→提交→填备注→等 pipeline→部署→确认），3 个节点 24 步
- 跑完 ETL 手写验证 SQL（COUNT、COUNT DISTINCT、非空率、跨层对比），每次都差不多但每次都要写
- 35 个节点统一改 cron 表达式，控制台操作 1 小时起

一天能专心写 SQL 的时间可能不到 30%。

**Skill 架构：三层结构**

用户通过 Chat/CLI 发起指令，AI Skill 智能助手拆分为三类功能模块，底层对接 MaxCompute（计算/存储）和 DataWorks（API/调度/集成），以及源库。

![AI Skill 架构图：用户层→AI助手层→平台能力层](https://mmbiz.qpic.cn/sz_mmbiz_png/zOKECicQfxFUOm50fssIUrx4RxBswPmJnHf1gZwM4Td1mWS281yS1ibhM3H9WU1mxL59Eu4gRRXib6ETVteSKppxYto5pcdqpr5VfH0oicKEYibQ/640)

**四大场景及核心命令：**

**1. 开发阶段**

- `/mc-schema ods_order_main` — 查看表结构（列数、分区、字段类型），不登控制台
- `/pg-query biz_db SELECT ...` — 跨库核对源数据，不装客户端、不配连接串
- `/mc-run-sql etl_dwd_order_detail.sql 20250405` — 本地 SQL 直接跑到 MaxCompute，自动替换日期变量，跑完输出行数分布

Skill prompt 核心：读取本地 SQL 文件，替换日期变量，去掉 SET 语句，调用 PyODPS execute_sql 执行，自动执行行数分布统计。

**2. 测试阶段（提效最明显）**

- `/mc-dq ads_report_daily 20250405` — 一行命令跑完：总行数、主键唯一性、核心字段非空率、跨层一致性、环比波动

DQ 检查规则：重复 >0 标注告警、空值率 >5% 标注告警、波动 >20% 标注告警。

关键价值：发现问题后在同一对话里直接追因（如追问"coupon_id 非空率为什么只有 32%"），形成"检查→发现异常→追因→确认预期"的闭环，这是脚本做不到的。

**3. 部署阶段**

- `/dw-deploy etl_dwd_order_detail etl_dws_order_agg etl_ads_report_daily` — 批量部署 3 个节点，8 步手动操作 × 3 = 24 步点击简化为 1 行命令
- `/dw-create-dijob pg_external new_schema ods_new_` — 创建整库同步任务，自动配置 26 张表映射、分区规则、调度参数
- `/dw-set-deps` — 配置依赖链
- `/dw-update-cron` — 统一调度频率
- `/dw-offline` — 清理失效节点

底层 API 链路：`update_file`（更新 SQL）→ `submit_file`（提交）→ `deploy_file`（部署）。

**4. 运维阶段**

- `/dw-status` — 全链路状态一览（DI 同步状态、各链路执行状态、数据新鲜度）
- `/dw-log etl_ads_report_daily 20250405` — 查看失败日志定位原因
- `/dw-rerun etl_ads_report_daily 20250405` — 修复后重跑

查日志→看原因→改 SQL→部署→重跑→验证，全在一个窗口里闭环。

**真实案例：三小时完成链路重建**

搜索服务的数仓链路需要从源表出发完整重建。ADS 表错误依赖了另一条链路的中间表，数据粒度和字段都对不上。

操作流程：
1. 分析：`/pg-query` + `/mc-schema` 快速遍历所有表结构，`/mc-query` 核对数据量和分布
2. 开发：编写 3 个 ETL SQL（约 900 行），`/mc-ddl` 批量建 12 张表
3. 验证：`/mc-run-sql` 逐层执行，`/mc-dq` 全套质量检查——百万级数据零膨胀零重复
4. 部署：`/dw-deploy` 批量部署节点，`/dw-set-deps` 配置依赖链，`/dw-update-cron` 统一调度频率，`/dw-create-dijob` 创建同步任务，`/dw-offline` 清理失效节点

![数据验证汇总结果：ODS 同步 48 张表全部成功，DWD=DWS=ADS 跨层一致性通过，主键唯一、核心字段非空率达标](https://mmbiz.qpic.cn/mmbiz_png/zOKECicQfxFWb5DT7udHia10MNjecjKxmWF2Ipe3G1Kv9lIdgRyyuxlza34ATVFiaJ8seR6eRpIvic3MAykjEfS7TOVPvJqMia2ue0HibaPPGqvTg/640)

**故障排查案例**

ADS 表停更 7 天导致下游搜索服务数据源断裂。传统方式需要在 DataWorks 控制台、节点管理、日志页面间来回跳，用 Skill 在一个对话窗口 3 秒定位根因：数据源类型注册为 RDS，实际应为 PostgreSQL。

**如何构建 Skill**

三步法：

1. 梳理高频操作 — 回想过去一周哪些操作每天都在重复（查表结构、查调度、部署节点、写验证 SQL）
2. 找到对应 API — DataWorks 和 MaxCompute 都有 OpenAPI（`ListInstances`、`CreateDIJob`、`SubmitFile`+`DeployFile`）
3. 写 Skill prompt — 核心就是一段 prompt：做什么、调哪些 API、怎么输出

建议从最常用的操作开始（作者第一个是 `/dw-status`），不需要一步到位。