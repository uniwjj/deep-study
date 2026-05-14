---
ingested: 2026-05-14
wiki_pages: [dataagent-semantic-layer]
---
# DataWorks DataAgent 语义层总结

## 一、从分享录音/PDF中提取的描述

### 许志（DataWorks 主讲人）所述语义层内容

1. "我们也内置了跟引擎层的连接，整个网络层包括**语义层**我们都会有连通的"
2. "两个 Agent 它不是说独立的产品，它有一个非常非常强大的**统一的上下文**"
3. "这两套 Agent 我们认为它是**懂数据的、懂你的代码的、懂我们整体的安全权限体系的**，所有的能力是可以共享的"
4. "上下文是可以**持久化下来的**，在 Claw 里边之前所有在 DataAgent 开发过的内容是可以被 Claw 读取到的"
5. "受控的执行内核，并且这个内核是可以被两种甚至多种模式**共享和使用**的"
6. "如果你确定了一个规则，那 AI 在确定规则的情况下它的执行能力还是非常非常强的"（指表名规范等）

### PDF 幻灯片中描述

> 数据语义层：统一的数据字典、术语映射和业务上下文

---

## 二、从数据开发 SKILL 中提取的内容

### 1. 节点类型体系（显式定义）

`references/nodetypes/` 目录下按类别组织了约 150+ 种节点类型，每种含：

- `command`：节点类型标识（如 `ODPS_SQL`、`HOLOGRES_SQL`、`FLINK_SQL_STREAM`、`DIDE_SHELL`、`DI`）
- `contentFormat`：代码格式（sql / shell / python / json / empty）
- 文件扩展名：`.sql`、`.sh`、`.py`、`.json`、`.vi`
- `datasourceType`：数据源类型（odps / hologres / flink / emr / mysql / clickhouse / starrocks）
- 运行时约束：代码大小上限、查询行数限制、超时默认值、并发度等

常用节点类型映射表（SKILL.md 显式列出）：

| 用途 | command | contentFormat | 扩展名 | datasource |
|------|---------|--------------|------|------------|
| Shell 脚本 | DIDE_SHELL | shell | .sh | — |
| MaxCompute SQL | ODPS_SQL | sql | .sql | odps |
| Python 脚本 | PYTHON | python | .py | — |
| 离线数据同步 | DI | json | .json | — |
| Hologres SQL | HOLOGRES_SQL | sql | .sql | hologres |
| Flink 流式 SQL | FLINK_SQL_STREAM | sql | .json | flink |
| Flink 批处理 SQL | FLINK_SQL_BATCH | sql | .json | flink |
| EMR Hive | EMR_HIVE | sql | .sql | emr |
| EMR Spark SQL | EMR_SPARK_SQL | sql | .sql | emr |
| Serverless Spark SQL | SERVERLESS_SPARK_SQL | sql | .sql | emr |
| StarRocks SQL | StarRocks | sql | .sql | starrocks |
| ClickHouse SQL | CLICK_SQL | sql | .sql | clickhouse |
| 虚拟节点 | VIRTUAL | empty | .vi | — |

### 2. 数据集成连接器（显式定义）

`references/nodetypes/data_integration/datasources/` 下共 48 个数据源目录，每个含 Source.md 和 Destination.md，定义：

**读写参数**：
- `datasource`：数据源名称（必须与 DataWorks 中注册的名称完全一致）
- `table`：表名
- `column`：列映射（JSON 数组，读写列数量必须对齐）
- `partition`：分区（MaxCompute 格式 `dt=${bizdate}`）
- `where`：过滤条件
- `splitPk`：切分键（用于并行读取）

**写入模式**：
- MySQL：insert / update（on duplicate key）/ replace（replace into）
- Hologres：replace / ignore / update
- MaxCompute：truncate（Insert Overwrite）/ 非 truncate（Insert Into）
- OSS：truncate / append / nonConflict
- Elasticsearch：index（upsert）/ update（部分更新）
- MongoDB：isReplace（以 `_id` 为替换键）
- Redis：string / list / set / zset / hash（每种独立配置）

**通道配置（settings）**：
- `speed.concurrent`：并发度 1-32，默认 3，分布式模式需 >= 8
- `speed.throttle`：是否限流
- `speed.mbps`：限流带宽
- `errorLimit.strategy`：Ignore / Tolerance / ZeroTolerance
- `errorLimit.record`：脏数据容忍上限
- `executeMode`：distribute（分布式）/ null（单机）
- `failoverEnable`：至少一次语义

**DIJob 顶层结构**（DI.md 定义）：
- `type` + `version` + `steps`（reader/writer）+ `order`（hops）+ `setting` + `extend`
- `extend` 块必含 `mode`、`__new__`、`formatType`、`resourceGroup`、`cu`

### 3. FlowSpec 规范（显式定义）

`references/flowspec-guide.md` 定义的标准 JSON 结构：

**顶层**：
- `version`：`"2.0.0"`
- `kind`：13 种枚举（`Node` / `CycleWorkflow` / `ManualWorkflow` / `Component` / `Resource` / `Function` 等）
- `spec`：实际内容

**节点关键字段**：
- `name`：节点名称，项目内唯一
- `id`：必须与 `name` 完全一致
- `script.path`：必须以节点名称结尾
- `script.runtime.command`：节点类型标识（必填，创建后不可变）
- `script.content`：代码内容
- `outputs.nodeOutputs[].data`：格式 `${projectIdentifier}.NodeName`，项目内全局唯一
- `dependencies[].nodeId`：自引用，必须为当前节点自身的 name
- `dependencies[].depends[].output`：上游节点的输出
- `datasource`：`{name, type}`，type 必须与 command 的 datasourceType 匹配
- `runtimeResource`：`{resourceGroup}`
- `trigger`：`{type, cron, startTime, endTime, timezone}`
- `recurrence`：Normal / Pause / Skip
- `rerunMode`：Allowed / Denied / FailureAllowed
- `rerunTimes` / `rerunInterval`：重试次数及间隔

### 4. 调度语义（显式定义）

`scheduling-guide.md` 中：

- **Cron 表达式**：6 位 `ss mm HH dd MM ?`
- **实例生成模式**：`T+1`（今天配置明天生成实例）或 `Immediately`（当天生成）
- **业务日期**：业务日期 = 运行日期 - 1
- **空跑实例**：非调度日生成的状态为"成功"的虚拟实例，不执行代码，用于衔接依赖
- **周期实例执行条件**：所依赖的上游实例均成功 + 自身调度时间已到
- **批量实例生成窗口期**：每天 23:30-24:00，此期间部署需到第三天生效

**调度时间示例**：
| 调度周期 | cron |
|---------|------|
| 天 | `00 00 03 * * ?` |
| 小时 | `00 00 */6 * * ?` |
| 分钟 | `00 */30 * * * ?` |
| 周 | `00 00 01 ? * MON` |
| 月 | `00 00 02 1 * ?` |

### 5. 依赖语义（显式定义）

`workflow-guide.md` 和 `flowspec-guide.md` 中：

- **Normal**：同周期标准依赖，等上游成功后执行
- **CrossCycleDependsOnSelf**：依赖自己上一周期的运行结果
- **CrossCycleDependsOnOther**：依赖其他节点上一周期的运行结果
- **根节点**：无上游依赖时，`dependencies[].output` 为 `${projectIdentifier}_root`（下划线分隔）
- **工作流内依赖**：A→B，A 声明 `outputs`，B 在 `dependencies` 中引用 `"${projectIdentifier}.A"`
- **跨工作流依赖**：同项目内使用 `${projectIdentifier}.other_wf_node`
- **跨项目依赖**：直接写 `"upstream_project_name.upstream_node_name"`
- **关键**：`dependencies[].nodeId` 为自引用（当前节点名称），`depends[].output` 为上游输出

### 6. 占位符机制（显式定义）

`flowspec-guide.md` 中：

- `${spec.xxx}`：spec.json 中按环境替换，值来自 dataworks.properties 中的 `spec.xxx`
- `${script.xxx}`：代码文件中按环境替换，值来自 dataworks.properties 中的 `script.xxx`
- `${projectIdentifier}`：outputs/dependencies 中的项目标识

**调度参数**：
| 变量 | 含义 | 示例值 |
|------|------|--------|
| `$yyyymmdd` | 业务日期(T-1) | `20260321` |
| `$[yyyymmdd]` | 运行日期(T+0) | `20260322` |
| `$[hh24]` | 运行小时 | `14` |
| `$[yyyymmdd-1]` | 运行日期-1天 | `20260321` |
| `$gmtdate` | 当前时间戳 | `20260322143000` |
| `${bizdate}` | 业务日期 | `20260321` |

### 7. 部署流水线状态机（显式定义）

`deploy-guide.md` 中：

**Pipeline 状态**：
- `Success`：部署成功（唯一成功状态）
- `Fail`：某阶段失败
- `Termination`：已终止（不是成功）
- `Cancel`：已取消

**Stage 类型**：
- `Build`：构建部署包，自动运行
- `Check`：检查，需手动推进
- `Deploy`：部署到生产，需手动推进
- `Offline`：下线，需手动推进

**流程**：`create-pipeline-run --type Online --object-ids <ID>` → 轮询 `get-pipeline-run` → `exec-pipeline-run-stage` 推进各阶段。

### 8. 工作流编排控制节点（显式定义）

- `CONTROLLER_BRANCH`：条件分支，`branch.branches[]` 定义条件表达式，按 Python 求值
- `CONTROLLER_JOIN`：分支归并节点
- `CONTROLLER_CYCLE`：do-while 循环，`maxIterations`（默认 128），`parallelism`（0=串行）
- `CONTROLLER_TRAVERSE`：for-each 遍历，`loopDataArray` 绑定上游集合
- `CONTROLLER_ASSIGNMENT`：赋值传递，Shell→echo / Python→print / SQL→SELECT 输出传递给下游

### 9. 运行时约束（显式定义）

- ODPS_SQL：代码最大 128KB，SQL 命令数 200，查询结果 10000 行/10MB
- PYODPS3：Python 3.7，本地数据上限 50MB/16CU，仅支持 print() 输出，不支持 logger
- DIDE_SHELL：不支持交互式语法，Serverless 最多 64 CU
- DI 并发度：1-32，默认 3，分布式需要 >= 8
- 节点超时：默认 4 HOURS
- 重试间隔：默认 180000ms（3 分钟）
- 批量 API 调用间隔：500ms 以上

### 10. 模板示例中的隐性惯例

以下出现在示例 SQL 代码中，SKILL 未将其定义为规则：

- 表前缀 `ods_`、`dwd_`、`ads_`、`dim_`
- `INSERT OVERWRITE TABLE ... PARTITION (dt='${bizdate}')`
- `WHERE dt = '${bizdate}'`
- `COALESCE(amount, 0)`
- `CURRENT_TIMESTAMP AS etl_time`
- `LEFT JOIN dim_*`
- `CASE WHEN status = 1 THEN 'completed' ELSE 'pending' END`

---

## 三、对比

| 来源 | 语义层描述 |
|------|-----------|
| **分享录音** | 统一上下文（代码+数据+权限+历史记录）、跨 Agent 持久化共享 |
| **PDF 幻灯片** | 统一的数据字典、术语映射和业务上下文 |
| **SKILL（显式）** | 150+ 节点类型体系、48 个数据连接器参数、FlowSpec 规范、调度语义、占位符机制、部署状态机、运行时约束 |
| **SKILL（示例暗示）** | ods_/dwd_/ads_/dim_ 分层前缀、分区覆写写法、审计字段写法、维度关联写法 |

**结论**：分享中语义层描述含糊，实物层面就是数据开发 SKILL 中编码的节点类型、连接器参数、FlowSpec 规范、调度规则等结构化知识。SKILL 中的示例代码暗示了数仓分层的命名惯例，但未将其上升为显式规则。
