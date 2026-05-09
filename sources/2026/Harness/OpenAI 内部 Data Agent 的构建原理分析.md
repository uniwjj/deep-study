---
ingested: 2026-05-09
wiki_pages: [big-data/openai-data-agent]
---
## OpenAI Data Agent

**文章信息**：
- **原文标题**：OpenAI 内部 Data Agent 的构建原理分析
- **来源**：https://mp.weixin.qq.com/s/CWsRW6D7A4qTZKQ67PH9kA
- **作者**：小皮大杰瑞
- **发表时间**：未知

---

**一段话总结**：

OpenAI 两名工程师用三个月（70% 代码由 AI 生成）构建的内部 Data Agent，服务 4000+ 员工，覆盖 600 PB 数据、7 万个数据集。其核心设计不是简单的 Text-to-SQL，而是在模型生成 SQL 之前，通过六层上下文体系注入业务语义，解决"语法正确但语义错误"的根本问题。六层上下文从物理元数据到实时仓库查询逐层递进，离线阶段统一向量化存储，查询阶段通过 RAG 按需拉取。表发现是最大技术难点——模型倾向于过早锁定候选表，系统通过 prompt engineering 要求模型在"发现模式"中停留更久。Memory 系统捕捉对话中的纠错信息并持久化，避免同类错误重复。Agent Loop 采用结构化多阶段执行，支持流式推理和用户中断。评估体系设定明确上线门槛：准确率 80%、一致性 70%。

---

**作者判断**：

作者认为 Data Agent 的核心挑战不在于 SQL 语法正确性，而在于语义正确性——找到正确的表和字段。原文原话："难的是如何找到正确的表、字段。以及如何正确地处理字段"。作者强调构建 Metadata pipeline 与 Data pipeline 同等重要，Data Agent 是"非标准的系统化工程，产品化需要大量的方案式实施"。最终结论："要设计一个好的 Data Agent，既要懂数据，更要懂 AI。"

---

**核心内容**：

**一、为什么不直接 Text-to-SQL**

直接让大模型生成 SQL 在 OpenAI 的规模（600 PB、7 万数据集）下面临结构性问题：模型缺乏对内部数据语义的理解。同名字段口径不同、术语定义因团队而异、dashboard 数据源可能已废弃——这类信息分散在内部文档、Slack 消息和特定人员的知识中，不在 schema 或模型训练数据里。OpenAI 的思路：**在生成 SQL 之前，先注入足够的业务上下文**。

![Data Agent 整体架构：多入口接收请求，通过离线知识库和在线数据源协作](https://mmbiz.qpic.cn/sz_mmbiz_png/hPzZyU2fe2DvnTwMvbksPtvsYCyq8R1PC31XgvibhLSoEprslvLKKTCxWmOibbwef9R9GHqkgAUHgSJjVCbhOcF3rmMS0dD9EsiazCEh7bVNzs/640)

**二、六层上下文体系**

![六层上下文从下到上：Table Usage → Human Annotations → Codex Enrichment → Institutional Knowledge → Memory → Runtime Context](https://mmbiz.qpic.cn/sz_mmbiz_png/hPzZyU2fe2Cc89IpeUfGETiaaofkXN2RVIVm3LeqmXRp8ClJNpV8kclMGs0glvYXYEjJlDjIWeQianaEjTnRR6hvpN7y4NYyNxV6MxWKHODpE/640)

1. **Schema 元数据与表血缘**：表名、列名、数据类型、上下游依赖关系（lineage）
2. **人工标注的专家描述**：数据团队维护的核心表和 dashboard 的业务含义、使用限制、常见误用
3. **Codex Enrichment**：每日异步 pipeline，用 Codex 读取 pipeline 代码，推断表的上下游、粒度、join key、ownership、相似关系，将隐含业务逻辑转化为可检索的结构化知识
4. **Institutional Knowledge**：从 Slack、Google Docs、Notion 提取数据相关上下文（如产品发布对指标的影响、术语的内部特定含义）
5. **Memory 系统**：持久化对话中的纠错信息（见第四节）
6. **实时仓库查询**：以上层级不足时，直接查仓库获取新表信息

六层上下文离线聚合为向量 embedding 存入向量数据库，查询阶段通过 RAG 仅拉取相关片段，使 7 万张表规模下的查询延迟可控。

**三、表发现：最大挑战**

在 7 万张候选表中定位目标是核心技术难题。模型的固有倾向是快速锁定一张"自认为正确"的表，不主动验证，导致后续分析全部建立在错误基础上。OpenAI 通过 prompt engineering 要求模型在"发现模式"中停留更久：收集多个候选表 → 比较粒度和定义差异 → 显式验证 → 再进入分析。

**非直觉发现**：注入更多上下文并不总是更准确。超过阈值后模型注意力分散，反而降低定位准确率。**上下文筛选质量的影响不低于覆盖范围。**

![上下文检索架构：左侧离线 RAG 嵌入各类知识数据，右侧实时语义搜索 + 精确文本检索](https://mmbiz.qpic.cn/mmbiz_png/hPzZyU2fe2CmgktBUK6cYfhJCVyT4A95lVIVKuTiaYwPhhibuDxKVVttD3zFAhibTYyricH6vCLN5fYIhfHVfOWmU4xRMKGqQNQLiawDiad2OK1Lc/640)

**四、Memory 系统**

RAG 解决的是从已有知识库中检索信息的问题，Memory 解决的是无法从 schema/文档/代码中推断的隐性约束——只有在实际查询中遇到问题并被纠正后才会产生。

典型案例：系统曾无法正确过滤实验数据，因为过滤条件依赖 experiment gate 配置中的特定字符串，该字符串不出现在任何 schema、文档或代码注释中。

设计要点：
- **触发机制**：用户主动纠正，或 Agent 自主识别学习点后发起保存提示
- **作用范围**：全局 Memory（所有用户生效）和个人 Memory（仅创建者生效）
- **可编辑性**：用户可手动创建、修改、删除

**五、Agent Loop**

不以单次生成回答，而是在结构化 loop 中逐步推进：意图理解 → 发现阶段 → 规划 → 执行 → 验证 → 回答。

三个关键设计：
- **实时流式推理**：中间步骤（候选表、选择依据、执行阶段）实时传输给用户
- **可中断性**：用户可在任意阶段介入调整方向，无需重新发起查询
- **Checkpoint 与自评估**：故障时从中断点恢复，任务结束时模型自我评估质量

**六、评估体系**

![评估流程：生成 SQL 与预期 SQL 比较 + 生成 SQL 结果与预期结果比较，双重维度综合评分](https://mmbiz.qpic.cn/sz_mmbiz_png/hPzZyU2fe2DTjm3u9JYDXwlt0thp9fFhozWR17ibzOVZUs2x8uz0bLFxJ07JK1CAHHu5ibx13ZlykWpXQgkTV9kyxRaHco1HQibOngQAMZ1jEw/640)

- **评估方法**：同时比较 SQL 语法和查询结果，两个维度信号输入 Evals grader 生成综合评分（语义等价的 SQL 可能语法不同，结果可能含额外列）
- **运行方式**：开发阶段持续运行检测回归；生产环境 canary 运行提前发现异常
- **上线门槛**：准确率 ≥ 80%，一致性 ≥ 70%，低于标准不进入生产

**七、核心思考**

1. **Data Agent ≠ Text-to-SQL**：语法正确已不是难点，语义正确才是核心挑战
2. **Metadata pipeline 与 Data pipeline 同等重要**：物理元数据、血缘、数据特征的采集；语义修正和增强；业务语义集成——都需要专门的 pipeline
3. **非标准工程**：企业内部特有的语义采集难以标准化，对 Agent 质量有直接影响
4. **需双重认知**：设计好的 Data Agent 既要懂数据工程，也要懂 LLM 原理和限制（context window 约束、Memory 设计、避免死循环的 Agent Loop）
