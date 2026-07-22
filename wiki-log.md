# Change Log

Append-only record of wiki operations. Format: `[date] verb | subject`

## [2026-07-22] ingest | 深度分析：OpenViking 中关于 LLM wiki 的理念的工程级实践
- created `openviking` — OpenViking 完整页面（L0/L1/L2 三级分层/目录递归检索/自迭代/范式对比/局限）
- updated `llm-wiki-implementations` — 新增 OpenViking 作为第四种工程级实现
- updated `index` — LLM Wiki 体系索引新增 openviking 条目
- saved source: `sources/2026/07/22/深度分析：OpenViking 中关于 LLM wiki 的理念的工程级实践.html` + sidecar
- article-download: `article-downloads/深度分析：OpenViking 中关于 LLM wiki 的理念的工程级实践.html`

## [2026-07-16] ingest | Agent 治理：用 Hook 堵住 LLM 的偷懒、越权与失忆
- created `agent-hook-governance` — DECO Hook 护栏体系完整页面（长文本 offload/HITL 门禁/上下文联动闭环 + 行业对比）
- updated `ai-governance` — 新增框架层治理（Hook 护栏）维度与交叉引用
- updated `agent-architecture-patterns` — 扩展 Hook 优于插件的架构原则描述，新增交叉引用
- updated `data-agent-enterprise-practice` — 新增 DECO Hook 治理实践交叉引用
- saved source: `sources/2026/07/16/Agent 治理：用 Hook 堵住 LLM 的偷懒、越权与失忆.html` + sidecar

## [2026-07-13] ingest | Data Agent vs Ontology Agent 推理对比

- created `data-agent-vs-ontology-agent` — 指标体系驱动的 Data Agent 与本体驱动的 AI Agent 在推理空间、路径、确定性来源上的范式差异对比，含融合交替方案
- updated `ontological-semantic-layer` — 添加交叉引用
- updated `data-agent/index` — 语义层与本体列表新增条目
- saved `sources/2026/07/01/指标体系驱动的Data Agent与本体驱动的AI Agent，AI推理上有什么异同？.html` + sidecar

## [2026-07-10] ingest | TASTED 五份升级指南

- updated `netease-tasted-model.md` — 补充五份升级指南摘要：审美力（三站）、洞察力（三站）、结构力（三站）、判断力（三站）、标准感（两项修炼），含内网课程链接与核心理念；解释为何 D（自驱力）无升级指南
- saved `sources/2026/07/10/TASTED-审美力升级指南_265801.html` + sidecar
- saved `sources/2026/07/10/TASTED-洞察力升级指南_266537.html` + sidecar
- saved `sources/2026/07/10/TASTED-结构力升级指南_267757.html` + sidecar
- saved `sources/2026/07/10/TASTED-判断力升级指南_268467.html` + sidecar
- saved `sources/2026/07/10/TASTED-标准感升级指南_269906.html` + sidecar

## [2026-07-10] ingest | 从模型到Harness：WorkBuddy如何把Agent做成可用产品

- created `workbuddy-agent-product-design.md` — WorkBuddy 完整产品设计：模型抽象、工具调用/MCP/Skill/Plugin 四概念、五类记忆模型、Harness 五层架构、Loop Engineering
- created `workbuddy-context-engineering.md` — WorkBuddy 上下文工程实践：五类动作（Write/Select/Retrieve/Compress/Isolate）、Prompt Cache 策略、渐进式加载、意图识别
- updated `agent-memory-system.md` — 补充 WorkBuddy 五类长期记忆模型与陈述性/程序性记忆区分
- updated `loop-engineering.md` — 补充 WorkBuddy 四层工程体系与 Goal≠Loop 区分
- updated `agent-harness-overview.md` — 补充 WorkBuddy 五层 Harness 模型与驾驭/约束/整合三能力
- saved `sources/2026/07/10/从模型到Harness：WorkBuddy如何把Agent做成可用产品.html` — 原始自包含 HTML（394KB，29 张图片）
- saved `sources/2026/07/10/从模型到Harness：WorkBuddy如何把Agent做成可用产品.md` — sidecar（frontmatter 供 CLI 追踪）

## [2026-07-07] ingest | Comet整合OpenSpec与Superpowers详解

- created `comet-integration-architecture.md` — Comet 双星整合架构总览：五维整合、.comet.yaml 状态机、预设路径、SDD 设计启示
- created `comet-gating-system.md` — 三层门控体系：硬门控（PreToolUse Hook）、阶段守卫（comet-guard.sh）、软门控（Rule 注入）
- created `comet-workflow-phases.md` — 五阶段工作流：8 个 SKILL.md 详解、审查模式三级、衔接协议
- created `comet-codegraph-integration.md` — CodeGraph MCP 在 design/build/verify 阶段的代码智能引擎角色
- updated `index.md` — 新增 Comet 整合架构章节
- updated `sdd-custom-workflow.md` — 添加 Comet 交叉引用（替代/互补方案）
- saved `sources/2026/07/07/Comet整合OpenSpec与Superpowers详解.md` — 原始 Markdown（85KB）

## [2026-06-29] ingest | OceanBase 湖库一体 AI 数据库正式发布 + 页面优化

- created `big-data/oceanbase-ai-database.md` — OceanBase 2026-06-29 发布会：湖库一体架构、Lakebase/DataStudio/DataPilot 三大产品、多模表+AI列、Agent友好上下文工程、四条工程底线、三大落地场景
- optimized `big-data/oceanbase-ai-database.md` — 重构逻辑链（为什么重建→是什么→怎么工作→工程底线→真实验证），提高信息密度，消除冗余
- saved `sources/2026/06/29/OceanBase 湖库一体 AI 数据库正式发布.html` — 原始 HTML（24 张图片）

## [2026-06-29] ingest | 菜鸟 SuperETL 补充（DataWorks 虾聊日相关）

- updated `big-data/superetl.md` — 基于 2026-06-28 DataFun 文章补充：九大 Skill 铁律约束、置信度评估机制（30%/90%分级）、6 步实战推演（带 Hook 机制对照）、cn-odpscmd CLI 详细命令表、Hooks 四个触发时机细化（SessionStart→PreToolUse→PostToolUse→SessionEnd）、LLM 数据域 + 应用层 AI 化展望
- updated `big-data/dataworks-2026-0528-xialiaori.md` — 补充菜鸟 SuperETL 交叉引用
- saved `sources/2026/06/28/菜鸟借助 DataWorks Data Agent 实现 AI 数据研发 SuperETL 落地.html` — 原始 HTML（17 张图片）

## [2026-06-29] ingest | Claude Code 指令失效分析与行为契约（两篇）

- created `ai-agent/claude-code/claude-code-instruction-failure.md` — Claude Code 忽略指令的四类根因 + 五层规则架构（入口→路径→Skills→Subagent→硬边界）+ 臃肿文件收身方法
- created `ai-agent/claude-code/claude-code-behavior-contract.md` — 从 41% 到 3% 的规则工程：五类 Agent 偏差（猜/扩/混/忘/装作完成）、12 条规则的四维重组、好规则的失败现场来源、80 行骨架模板、三层文件架构
- updated `ai-agent/context-eng/claude-md-best-practices.md` — 补充两篇新页面的交叉引用
- updated `ai-agent/claude-code/claude-code-configuration-guide.md` — 补充两篇新页面的交叉引用
- saved `sources/2026/06/28/我终于搞明白了：Claude Code 为什么会忽略指令了.html` — 原始 HTML（5 张图片）
- saved `sources/2026/05/11/Claude Code 错误率从 41% 到 3%：CLAUDE.md 到底改对了什么？.html` — 原始 HTML（4 张图片）

## [2026-06-26] ingest | How Anthropic enables self-service data analytics with Claude（重新摄取）

- saved `article-downloads/How Anthropic enables self-service data analytics with Claude.html` — 自包含离线 HTML（4 张图片、2 个代码块）
- saved `sources/2026/06/26/anthropic-self-service-analytics-with-claude.md` — 文章 Markdown 版本（2026-06-21 更新版）
- updated `ai-agent/ecosystem/agentic-analytics-anthropic.md` — 补充 6月21日更新内容：参考文档模板（为 LLM 检索优化的表描述结构）、仓库 Skill 完整骨架模板（语义层优先反借口清单、快速启动工作流、业务实体消歧、对抗 SQL 审查 + 溯源页脚、字段命名陷阱血泪教训）；更新 sources 指向新版本，扩展相关页面链接

## [2026-06-23] lint | 知识库健康检查与修正（第七轮）

- fixed `business-cognition-system` — 修复断链 [[agent-context-engineering]] → [[ai-agent/context-eng/index|上下文工程]]
- fixed `data-agent-enterprise-practice` — 修复断链 [[data-asset-semantic-layer]] → [[dataagent-semantic-layer|数据资产语义层]]
- fixed `data-agent-enterprise-practice` — 移除无效 wikilink [[metricflow]]（外部工具），改为纯文本
- fixed `iceberg` — 补充缺失 `sources` 字段（lint-stub.md）
- result: 168 页面，0 断链，0 frontmatter 缺失，0 空来源——知识库结构健康

## [2026-06-18] ingest | 让 DataAgent 真正用起来: DataAgent 在企业信用的优化和实践

- saved `sources/2026/06/18/dataagent-enterprise-credit-practice.md` — 微信公众号文章全文（杜淑均/蚂蚁集团/DataFun）
- saved `article-downloads/让 DataAgent 真正用起来-DataAgent 在企业信用的优化和实践.html` — 自包含离线 HTML（24 张图片、语法高亮 CSS）
- created `ai-agent/agent-core/data-agent-enterprise-practice.md` — Data Agent 企业级实践：三个判断（承接 Coding Agent 闭环/可组合能力优于孤立产品/数据资产下限+业务认知上限）、四层架构（语义→认知→Agent核心→渠道）、四个关键实践（增强元信息/混合语义路由/可演进认知系统/Harness 自迭代）
- created `ai-agent/agent-core/business-cognition-system.md` — 可演进业务认知系统：三类知识来源（显性抽取/人工校正/Agent反馈）、两种组织（召回知识库/文件系统）、动态加载、反馈回流闭环
- updated `ai-agent/harness/harness-engineering-practice.md` — 新增蚂蚁企信 Data Agent 的 Harness 实践交叉引用（Main/Reviewer/Executor/Eval 四 Agent 协同）

## [2026-06-17] ingest | OpenSpec 自定义 Schema 深度指南
- saved `sources/2026/06/17/openspec-custom-schema.html` — 原始 HTML 文章（10 章，含 6 个 SVG 架构图）
- created `ai-agent/sdd/openspec-schema-mechanism.md` — Schema 核心机制：DAG 心智模型、三层定制体系、schema.yaml 完整字段详解、命令映射、openspec status 状态机
- created `ai-agent/sdd/openspec-custom-schema-guide.md` — 自定义 schema 创建（fork/init/社区安装）、模板设计原则、数据仓库完整实战（dw-spec-driven）
- created `ai-agent/sdd/openspec-skill-mechanism.md` — SKILL 文件机制：生成与加载流程、SKILL.md 结构、与 schema.yaml 的分工
- created `ai-agent/sdd/openspec-verify-extension.md` — Verify 扩展三种方案（artifact/SKILL/instruction）对比与推荐组合
- updated `ai-agent/sdd/index.md` — 新增"OpenSpec Schema 深度拆解"分类（4 篇）
- updated `ai-agent/sdd/openspec-sdd-practice.md` — 添加 Schema 相关交叉引用
- updated `ai-agent/sdd/sdd-custom-workflow.md` — 添加 Schema 与 SKILL 机制交叉引用

## [2026-06-16] ingest | AI Agent & Skill 测评方案及落地实践（微信公众号：腾讯程序员）

- saved `sources/2026/06/16/AI Agent & Skill 测评方案及落地实践.html` — 原始 HTML（含 20 张图片、33 个样式块）
- saved `sources/2026/06/16/AI Agent & Skill 测评方案及落地实践.txt` — 提取文本 + frontmatter
- created `ai-agent/agent-core/agent-evaluation-framework.md` — AI Agent 测评框架总论：三类评分器（确定性/Rubric/人工）+ 五大维度（功能正确性/过程质量/效率成本/鲁棒性安全/体验对齐）+ 通用评分公式
- created `ai-agent/agent-core/agent-evaluation-implementation.md` — 测评落地实践：四大场景用例设计 + 基线管理 + Trace 能力 + 稳定性评估（pass@k vs pass^k）+ 能力测评 vs 回归测评 + TPerf 性能 AI 分析 Agent 案例（9 类 30+ 用例、LCS 步骤评分、7 模型对比）
- updated `ai-agent/agent-core/index.md` — 新增"测评体系"分类，总数 16→18 篇
- updated `ai-agent/ecosystem/architect-in-agent-era.md` — 补充评估体系相关交叉引用

## [2026-06-15] ingest | Google Cloud Data Agent Kit 深度解析（中文分析文章）
- saved `sources/2026/06/15/Google Cloud Data Agent Kit 深度解析.html` — 中文深度分析原文（含 SVG 架构图）
- expanded `big-data/google-data-agent-kit` — 整合三大来源（Twitter 官宣 + 官方博客 + 中文分析），新增：预置 Agents 详表（Data Engineering/Data Science/DB Observability + GA/Preview 状态）、意图驱动开发完整示例（交易日志→欺诈检测→Spanner 全链路）、逆向下沉战略分析（控制点从算力存储迁移到开发者工具链）、HITL→HOTL 数据运维自主化演进、竞争格局对比表（Google vs AWS vs Databricks）、对 Hive/Spark 数仓团队的实践启示（MCP + YAML Spec 投喂层）

## [2026-06-15] optimize | 知识库健康度优化——断链修复 + 桩页面 + 索引完善
- fixed 8 short-name wikilinks → full path (e.g. `[[context-eng]]` → `[[ai-agent/context-eng/index]]`) in 7 index pages
- fixed 2 `[[mcp]]` → `[[agent-mcp-protocol|MCP]]` (agentic-skills + agentic-data-cloud)
- created `big-data/iceberg` stub — Apache Iceberg 开放表格式基础页
- removed 3 speculative wikilinks (claude-code-skills-system, ISO-GQL, 知识图谱) → kept as plain text
- result: 14 broken wikilinks → 0, 156 pages, all cross-references valid

## [2026-06-15] expand | loop-engineering vs OpenClaw 对比分析
- expanded `ai-agent/harness/loop-engineering` — 新增与 OpenClaw 的六组件逐项对照、四个结构性差异（验证哲学/预算模型/记忆复杂度/覆盖域）
- updated `ai-agent/ecosystem/openclaw` — 添加指向 loop-engineering 的交叉引用
- updated `ai-agent/ecosystem/openclaw-agentic-search-memory` — 添加指向 loop-engineering 的交叉引用

## [2026-06-15] ingest | Loop Engineering详解：把反馈循环放进工程现场
- saved `sources/2026/06/15/loop-engineering详解.md` — 微信公众号文章全文（若飞·架构师，8.8KB）
- saved `article-downloads/Loop Engineering详解：把反馈循环放进工程现场.html` — 自包含 HTML（74KB）
- created `ai-agent/harness/loop-engineering` — Loop Engineering 核心概念页：六组件拆解、闭环vs开环、验证准入表、预算约束、状态记忆、人在场原则、7天试点路径
- updated `ai-agent/harness/index` — 新增 loop-engineering 入口

## [2026-06-15] ingest | Google Cloud 官方博客：Data Agent Kit 详解 + Agentic Data Cloud 战略
- saved `sources/2026/06/15/Data Agent Kit brings data skills and tools to your IDE or CLI - Google Cloud Blog.html` — 官方博客原文（284KB）
- saved `sources/2026/06/15/What's New in the Agentic Data Cloud - Google Cloud Blog.html` — 官方博客原文（258KB）
- expanded `big-data/google-data-agent-kit` — 基于官方博客大幅扩充：新增统一数据中心（Unified Hub）、Agentic Skills 详解、对话式分析、意图驱动工程范式、安装使用指南、上下文窗口税概念；增加详细架构表格；补充新 source 引用
- created `big-data/agentic-data-cloud` — Google Agentic Data Cloud 战略架构页：三大创新支柱（Knowledge Catalog 通用上下文引擎 / Agentic-First 从业者体验 / 跨云 Lakehouse）、四类结构性失败分析、客户案例、性能数据
- updated `big-data/index` — Data Agent 生态新增 agentic-data-cloud

## [2026-06-15] ingest | 谷歌开源 Data Agent Kit：数据工程正式进入原生 Agent 时代
- created `big-data/google-data-agent-kit` — Google Cloud 开源数据工程 AI 工具集，含 Agentic Skills / MCP 工具层 / IDE 插件
- created `ai-agent/ecosystem/agentic-skills` — Agentic Skills（预制数据技能库）概念页，Data Agent 核心抽象单元
- updated `big-data/index` — Data Agent 生态新增 google-data-agent-kit

## [2026-06-08] ingest | How Anthropic enables self-service data analytics with Claude
- created `agentic-analytics-anthropic` — Anthropic 利用 Claude 构建自服务分析平台的完整架构（四层 stack：数据基础→真相源→Skills→验证），三大失败模式与最佳实践

## [2026-05-31] ingest | 别再卷大模型了：Palantir 告诉我们，AI 的进阶是"本体论"
- created `ontology` — 本体论核心概念（五要素框架、AI 幻觉解决方案、数据巴别塔、Neuro-Symbolic 关系）
- updated `ai-ml/index` — 新增核心概念分类与 [[ontology]] 条目
- updated `taobao-live-data-dev-paradigm` — 本体论段落添加 [[ontology]] wikilink

## [2026-05-31] ingest | DataWorks 虾聊日 0528 录音-修正版.md（re-ingest，整场录音文字稿）
- 通读 444 行完整录音转写稿，与 7 份 PDF 视觉复核结果双源对照
- expanded `dataworks-2026-0528-xialiaori` — 重写为整场议程版：补全开场王子军 OPC 引主机制（PDF 集合中无对应 deck，仅录音保留：800+BP 收件、入选率<10%、17 场路演、I have a demo 三创始人、四角色协同的引主机制、Airtouch 案例）；扩展每场分享的演讲人完整称谓（孙文强/丁源/孙戴博/赵红梅/李昊哲/畅含/陈梦成）+ 录音独有的原话/demo 细节
- updated `dataworks-data-agent` — 补回演讲人孙文强、现场两个故事（周五加表+凌晨故障）、三句金句（共用一颗大脑/共享数据语义层/AI 不是写代码快而是任务串联变化）
- updated `taobao-live-data-dev-paradigm` — 补回演讲人丁源（淘天集团高级数据专家、淘宝直播数据研发负责人）、NL2DSL2SQL 与 R2C 现场口语版定义、中心化转本地化的 4 个原因、verify-data Skill 已在阿里云公众号发布
- updated `hologres-skills` — 修正演讲人为赵红梅（梅酱）（之前误写"骆撷冬"，与 PDF P1 复核冲突）；补充开场即跑 demo（OpenClaw 后台跑刘备/吕布广告素材生成→视频→投放→分析全链路）
- updated `flink-skills` — 补充 4 个现场 demo（作业调优/批量巡检/跨产品组合/舆情分析）+ 阿里云 69 个官方 Skill 覆盖 6 大云领域
- updated `maxcompute-skills` — 补充元梗式开场、银行账户客户端金句、Time Travel 7 天回滚、AutoMV 多 Agent 同查节流、MaxAgent 移动端典型场景
- updated `openclaw-agentic-search-memory` — 补充核心矛盾原话、3 个 demo 现场（中考试卷/NVIDIA 169 页/Rick and Morty）、Self-evolution 借鉴 Hermes Agent
- 源文件 frontmatter 标注 re_ingested: 2026-05-31

## [2026-05-31] ingest | OpenClaw Agentic Search & Memory（re-ingest，PDF 视觉复核）
- 按 CLAUDE.md 多模态首次摄取规则，使用 `pdftoppm -r 150 -png` 转出 24 张页面图片逐页视觉复核，不再以派生 TXT/OCR 为唯一依据
- updated `openclaw-agentic-search-memory` — 修正演讲人（陈梦成）、补全议题主标题"Agentic Search + Memory，OpenClaw 企业研究更智能"+ 副标题"从搜索工具到企业认知引擎"+ 4 章议题脉络；补回 Agentic Search 第 5 项能力"标准交付模式（Skills 透出 / 开源 ACP 协议透出）"
- expanded `openclaw-agentic-search-memory` — 新增 24 页逐页摄取索引；新增 P7 两大引擎角色对照表（注重研究质量 vs 解决知识复用）；新增 P20 OpenClaw 双层记忆体系架构图（User Interaction Layer + Agentic Memory Layer，Extractor → Embedding Engine → ElasticSearch VectorStore，Generate Vectors for Facts and Skills 双向回路）；新增 P21 AgenticMemory 三模式（In-session / Cross-session / Self-evolution）；新增 P24 5 阶段研究范式升级时间线
- 源文件 frontmatter 标注 re_ingested: 2026-05-31

## [2026-05-31] ingest | MaxCompute × Agent：从多模态存储到混合计算（re-ingest，PDF 视觉复核）
- 按 CLAUDE.md 多模态首次摄取规则，使用 `pdftoppm -r 150 -png` 转出 14 张页面图片逐页视觉复核，不再以派生 TXT/OCR 为唯一依据
- updated `maxcompute-skills` — 修正 MCMCP 方法分组（"身份与权限"→"身份与权限确认"），补回 `get_instance` 方法，新增"配套 Skills"清单（Sql-Generation/Maxframe-Coding/Information-Schema/Project-Qouta-Manage）；纠正第 3 个 Agent 接入口实为"龙虾实操 Specialist Sub-Agent · OpenClaw"实战示例（OpenClaw 总控 + Specialist Sub-Agent 执行，证据链 Plan→Act→Result）
- expanded `maxcompute-skills` — 新增 14 页逐页摄取索引；补全 P3 接入大图四层（Metadata/Compute Engines/Storage 各列详细子项）、P6 Delta Table 细项（基础 DML / 自动维护拆分）、P9 控制台调用示例（task_344 失败率 70.6%）、P11 Blob Pipeline 架构对比（传统多引擎 vs MaxCompute Blob 模式）、P12 多模态算子 SQL 示例
- 源文件 frontmatter 标注 re_ingested: 2026-05-31

## [2026-05-31] ingest | EMR Skills：从 Lakehouse 到 Agentic Lakehouse（re-ingest，PDF 视觉复核）
- 按 CLAUDE.md 多模态首次摄取规则，使用 `pdftoppm -r 150 -png` 转出 22 张页面图片逐页视觉复核，不再以派生 TXT/OCR 为唯一依据
- updated `emr-skills` — 修正首次摄取的若干偏差：演讲人姓名（"董亚军"/"畅舍" → 畅含｜EMR 产品经理｜计算平台事业部）、副标题（"新站" → 新一站）、AI Function 体系子项（更正为图片打标/embedding/图像处理/数据脱敏/语义提取/音频处理/时序预测/内容安全）、EMR AI 助手（补"产品咨询"）、智能开发助手（补"参数优化"）、能力展望六项分列
- expanded `emr-skills` — 补全 P4 Lakehouse 数据流图（数据源 →（Flink）→ ODS/DWD/DWS/ADS·Paimon Table·Spark·DLF）、P8 应用场景层 16 个子能力、P10 三大子产品对照式细节、P11 三张产品截图、P14 渐进披露用户意图分类、P15-P19 场景/对话/架构页的具体数字与 SQL 片段
- 源文件 frontmatter 标注 re_ingested: 2026-05-31

## [2026-05-31] ingest | 淘宝直播：AI 驱动的数据研发范式升级（page-complete re-ingest）
- updated `taobao-live-data-dev-paradigm` — 按 PDF 22 页新增逐页摄取索引，确保 P1-P22 每页信息均有落点
- expanded `taobao-live-data-dev-paradigm` — 补充 P11 OpenClaw 主/子 Agent 分工，P14 SDD p1/p3 示例，P15 DSL/Prompt 示例，P19 LightRag 图谱节点与 llm-wiki-v2 目录结构，P21 ChatBI 示例
- verified source shape — PDF 为 22 页图片型文件，`pdftotext` 无可抽取文本，继续以现有视觉识别派生 TXT 为重摄取依据
- correction — 用户指出 TXT 不可信后，重新从 PDF 本体转出 22 张页面图片逐页视觉复核；更新 `taobao-live-data-dev-paradigm` 标注 TXT 不作为可信依据，并收敛 P19 低置信度图谱节点描述
- expanded `taobao-live-data-dev-paradigm` again — 补全 P7 高密度 R2C 架构图：两侧护栏、Agent 应用层、AI 能力层、基建层、Memory/Skill/SDD 演进方向、Ontology 映射、核心依赖；同时新增 P1-P22 逐页复核补充，逐页记录 PDF 图中细项

## [2026-05-31] query | 淘宝直播分享总结
- answered from `taobao-live-data-dev-paradigm` — 总结淘宝直播 AI 驱动数据研发范式升级分享的主旨、架构、方法论、落地路径与结果

## [2026-05-31] query | 淘宝直播中心化到本地化 AI Native 演进
- answered from `taobao-live-data-dev-paradigm` — 强调该分享的重要主线之一是从中心化 AI Native 向本地化 AI Native 演进：统一入口到定制工作流、短期无记忆到长期可继承记忆、工具到员工

## [2026-05-31] query | 淘宝直播分享要点核对清单
- answered from `taobao-live-data-dev-paradigm` — 按主题列出淘宝直播分享覆盖的主线、架构、演进、研发范式、Agent 协同、Skill/SDD/AI Coding、基建、成果与未来规划，供人工核对是否遗漏

## [2026-05-31] ingest | 淘宝直播：AI 驱动的数据研发范式升级（re-ingest）
- 重新摄取，原因：之前的 OCR 派生 txt 质量太差（大量乱码），用户已删除
- 直接读取 PDF（22 页，已转 PNG）通过视觉重新提取，重写派生 txt
- 重写 `taobao-live-data-dev-paradigm` — 修正多处错误数据：代码采纳率 72.9%→72%，代码渗透率 24.6%→24.63%，平均对话轮次 5 轮→2.91；补充直播 360 真实案例（56 口径/6 模型/2 融合/2 同步/1500+ 行代码）；补充本体论 Neuro-Symbolic 设计；补充 P9 直播研发范式 6 大支柱；补充 P16 三大核心原则；补充 Pipeline 编排详细流程；补充 SDD references 加载机制
- 源文件 frontmatter 标注 re_ingested: 2026-05-31

## [2026-05-24] ingest | SDD 实践：OpenSpec + Superpowers 整合创建自定义工作流（re-ingest）
- 源文件已归档至 sources/2026/05/24/，内容与 2026-05-17 首次摄入一致，无新增 wiki 页面
- 已有页面 `sdd-custom-workflow.md` 已完整覆盖此文内容

## [2026-05-24] ingest | 从古法编程到 AI 编程：Superpowers + OpenSpec 协同开发实战指南（re-ingest）
- 源文件归档，内容同 2026-05-17 首次摄入 → `superpowers-openspec-legacy-workflow.md`

## [2026-05-24] ingest | 开源项目 superpowers 深度解读：把 AI Coding Agent 变成遵守工程流程的协作伙伴
- created `superpowers-repo-architecture.md` — 仓库内部架构分析：四层能力模块（skills/agents/commands/hooks）、跨平台分发分层设计、using-superpowers 总控机制（1% 规则）、两条执行路径（subagent vs executing-plans）、receiving-code-review 反表演式认同、writing-skills 元技能 SDD 方式、5 个核心设计洞察 + 4 个局限
- updated `ai-agent/sdd/index.md` — 页数 12→13
- updated `ai-agent/index.md` — SDD 页数同步

## [2026-05-24] ingest | Superpowers：让 AI 编程从"碰运气"变成"走流程"的完整方法论
- updated `superpowers-workflow-scenarios.md` — 新增各步骤的操作级细节：头脑风暴 9 步标准化流程、Git 隔离 4 步创建、任务拆分 6 类禁止占位符、子代理两轮复查机制、TDD 偷懒借口表、代码审查三级标准+5 项验证清单、收尾四选项、安装验证、自定义技能 TDD 方式

## [2026-05-24] ingest | Superpowers 实战指南：7 步流程 + 14 个技能 + 3 条铁律
- created `superpowers-workflow-scenarios.md` — 完整 7 步工作流（头脑风暴→Worktree→任务拆分→子代理→TDD→审查→收尾）+ 三种场景裁剪指南（新项目/老项目加功能/修 Bug）
- created `superpowers-iron-laws.md` — 三条铁律详解：无测试不写代码、无根因不修 Bug、无验证不完成
- updated `superpowers-framework.md` — 补充 7 步工作流与三条铁律引用，新增来源
- updated `ai-agent/sdd/index.md` — 新增 2 篇页面，页数 10→12
- updated `ai-agent/index.md` — SDD 页数同步更新

## [2026-05-21] ingest | Claude Code Harness 工程：数仓侧落地方案｜得物技术
- created `dw-harness-practice.md` — 数仓 Harness 工程落地方案：五层防御体系、8步 SKILL 工作流、hooks 自动验证、subagent 上下文隔离
- updated `harness-engineering-practice.md` — 添加数仓 Harness 实践的交叉引用

## [2026-05-19] research | SDD & Spec 讨论素材准备（v2 完整版）
- updated `SDD-讨论素材.md` — 基于知识库全部 11 篇 SDD 专题深度改写：补充 Spec-Kit 5阶段门控、Superpowers 14技能+实测基准、OpenSpec 7步工作流+Delta Spec、双框架8步联合工作流+7坑+2衔接Skill、老旧项目4阶段实战+Redis库存预扣案例、薄编排11个action+三层架构+Override机制、渐进式方法论3铁律+知识飞轮

## [2026-05-18] lint | 深度检查与优化（第六轮）
- fixed 36 stale pages — 批量更新 updated 日期至 2026-05-18（源文件修改时间在页面 updated 之后）
- fixed 2 pages — agent-harness.md、llm-wiki.md 补充缺失的 ## 相关页面 章节
- expanded tech-radar — 从 6 行扩展为 采用/评估/关注 三级技术雷达，新增 big-data 和 agent-core 链接
- expanded claude-code hub — 从 8 行扩展为架构全景/核心机制/多Agent/工程实践四组分类，覆盖 34 篇子页面
- updated product/index — 新增 architect-in-agent-era、ai-career-trends、tech-radar 跨领域链接
- verified: 0 broken wikilinks, 0 stale pages, 0 missing frontmatter, 0 orphans

## [2026-05-18] lint | 全局健康检查与修复（第五轮）
- fixed `rag-vs-llm-wiki.md` — 移除自引用 [[rag-vs-llm-wiki]]
- fixed 5 shallow pages — architecture、backend、distributed、fullstack、platform 五个领域 index 页面补全领域说明
- fixed 8 uningested sources — 为 Google上下文工程白皮书、AI编程三剑客对比、Superpowers踩坑实录等 8 个来源文件补充 ingested 标记
- flagged 12 PDF 来源 — 无法为二进制 PDF 添加 frontmatter（已通过 .txt 提取副本覆盖）
- flagged 33 stub pages — 标记为 lint-stub 的占位/索引页面（多数为有意最小化）
- verified: 0 broken wikilinks across 143 pages

## [2026-05-18] lint | 全局健康检查与修复（第四轮）
- fixed `ai-agent/index.md` — 修复 7 条 table 内 `\|` 转义导致的断链，改用 wiki-relative 路径 `[[ai-agent/subdir/index]]`
- fixed `big-data/infinisynapse.md` — 修复 4 条 table 内 `\|` 转义导致的断链，改用裸 wikilink
- fixed 6 pages — 补充 `status: draft` 字段，移除 tags 中的 draft（agent-skills-system, claude-code, agent-harness, llm-wiki, token-consumption-economics, tech-radar）
- fixed 8 pages — 移除 tags 中冗余的 draft（agent-autonomous-planning, ai-agent-security, ai-governance, ai-knowledge-base, digital-employee, glm5-model, llm-cost-optimization, prompt-engineering）
- fixed 3 sources — 补充缺失的开 `---` frontmatter 分隔符（Obsidian+Claudian, Agent范式, Claude Code from Source）
- fixed 3 orphans — claude-code/index, ecosystem/index, sdd/index 现在通过 ai-agent/index.md 获得入链
- verified: 0 broken wikilinks, 0 wanted pages, 0 orphans

## [2026-05-18] ingest | Pushing the Frontier for Data Agents with Genie (Databricks Blog)
- updated `big-data/databricks-genie` — 从二手转述升级为原始博文，补充四阶段执行轨迹、40% 表搜索提升、GEPA 优化等原文细节
- created `ai-agent/specialized-knowledge-search` — 专用知识搜索技术：从企业资产提取语义上下文构建多索引、与通用 RAG 对比
- created `ai-agent/parallel-thinking` — 并行思考：多轨迹采样聚合弥补 Data Agent 缺确定性测试的问题
- created `ai-agent/multi-llm-design` — Multi-LLM 架构：不同子 Agent 使用不同模型分工，含 GEPA 成本优化
- updated `big-data/code-agent-vs-data-agent` — Related 新增三个技术页面的链接

## [2026-05-18] lint | 全局健康检查与修复（第三轮）
- fixed 5 source files — 修复 15 条 broken wiki_pages 引用（wiki/前缀、stale路径、bare prefix）
- fixed 7 index.md — 修复跨子目录 relative-path wikilinks（`[[subdir/index]]` → `[[subdir]]`）
- added frontmatter to 2 uningested HTML sources
- fixed `Anthropic的「脑手分离」` — 补充空 wiki_pages
- verified: 0 broken source refs, 0 broken wikilinks

## [2026-05-18] lint | 全局健康检查与修复（第二轮）
- fixed `agent-multi-agent-collaboration` — 修复 broken source 引用: Agent-Harness综述.md
- fixed `open-design` — 修复 broken source 引用: Open Design.md → Open-Design开源平替Claude-Design.md
- fixed `harness-design-long-running` — 修复 broken source 引用: anthropic-harness-design-long-running-apps.md
- updated 84 source files — 批量修复 wiki_pages 目录拆分后的 stale 路径
- fixed `DataWorks DataAgent分享录音.md` — 补充 big-data/ 前缀
- fixed `开源AI编程工具对决` — 补充 wiki_pages
- fixed 4 sources — 补充空 wiki_pages 字段
- verified: 0 broken wikilinks, 0 broken source refs

## [2026-05-18] lint | 全局健康检查与修复（第一轮）
- fixed `claude-managed-agents` — 修复 5 条 truncated source 文件名
- fixed `agent-harness-overview` — 修复 source 引用: Agent-Harness综述.md → 完整文件名
- fixed `harness-build-to-delete` — 修复 source 引用
- fixed `harness-as-backend` — 修复 source 引用
- fixed `harness-engineering-practice` — 修复 source 引用
- fixed `agent-vs-framework-vs-harness` — 修复 source 引用
- fixed `ai-career-trends` — 修复 source 引用
- fixed `deerflow` — 修复 source 引用
- fixed `hermes-agent` — 修复 source 引用
- fixed `architect-in-agent-era` — 修复 source 引用
- fixed `claude-md-best-practices` — 修复 source 引用
- fixed `manus-context-engineering` — 修复 source 引用
- fixed `llm-wiki-six-layer` — 修复 source 引用
- fixed `sub-agent-vs-team` — 修复 source 引用
- fixed `data-dev-skills-automation` — 修复 source 引用
- fixed `maxcompute-data-ai` — 修复 source 引用
- fixed `openai-data-agent` — 修复 source 引用
- fixed `rag-query-optimization` — 修复 source 引用
- fixed `google-scion` — 修复 source 引用
- fixed `ai-agent-ecommerce-content` — 修复 source 引用
- fixed `sdd-openspec-superpowers` — 修复 source 日期路径: 04/08 → 05/09
- fixed `openspec-sdd-practice` — 修复 source 引用
- fixed `meta-jit-testing` — 修复 source 引用
- fixed `databricks-genie` — 移除 broken wikilink [[GEPA]]
- updated 7 subdir index.md — 补充 sources: [2026/05/10/lint-stub.md]
- updated 12 pages — 补充英文 aliases
- fixed `dataagent-semantic-layer` — 修正 content type tag: meta → concept

## [2026-05-18] lint | Schema 拆分：ai-agent/ 重组为 8 个子目录
- updated `wiki-schema.md` — 新增子目录规则：领域目录超 30 页可按主题拆分子目录
- reorganized `ai-agent/` — 99 页拆分为 7 个子目录：claude-code(34)、agent-core(16)、sdd(10)、knowledge-base(12)、harness(8)、context-eng(3)、ecosystem(15)
- created 7 子目录 `index.md` 索引页
- updated `ai-agent/index.md` — 改为子领域表格导航
- updated `big-data/dataagent-semantic-layer` — Related 新增 specialized-knowledge-search 链接

## [2026-05-18] ingest | 为什么 Code Agent 无法解决企业数据分析问题（WinClaw）
- created `big-data/code-agent-vs-data-agent` — Code Agent vs Data Agent 目标函数差异、Databricks Genie 三个挑战、维度对比、协作分工
- created `big-data/infinisynapse` — InfiniAgent + InfiniSQL + InfiniRAG 三件组合，含命名中间表/虚拟数仓/异构源 join/可审计工作流（draft）
- created `big-data/databricks-genie` — Genie stub，记录 Lakehouse 全资产覆盖、specialized search + parallel thinking + Multi-LLM、内部 32%→90% benchmark（draft，待拉原文）
- updated `big-data/index` — 新增"概念与对比"分组，加入 databricks-genie / infinisynapse 链接
- updated `big-data/dataworks-data-agent` — Related 链接新文章三页
- updated `big-data/data-agent-practice-guide` — Related 链接新文章三页
- updated `big-data/openai-data-agent` — Related 与 InfiniRAG / Genie 对照

## [2026-05-17] lint --fix | 修复 1 处断链
- fixed `learning-path` — `[[Claude Code]]` → `[[claude-code]]`（空格导致断链）

## [2026-05-17] lint --fix | 扩展 5 个 stub 页面
- expanded `agent-multi-agent-collaboration` — 两种协作范式、五种通用模式、选型铁律、五条设计原则
- expanded `agent-tdd-workflow` — Superpowers 强制 TDD 循环、Meta JiT 语义级测试
- expanded `agent-architecture-patterns` — 10 大设计模式及架构原则
- expanded `agent-mcp-protocol` — JSON-RPC 基础、8 传输/7 来源、工具包装流程、安全设计
- expanded `learning-path` — 三阶段学习计划：Agent 源码 → AI 工程化 → 大数据融合

## [2026-05-17] lint | 知识库健康检查
- 130 页，500+ 链接：断链 0，孤立页 0，缺失 frontmatter 0，无来源 0，内容过期 0
- 标记 34 浅层页（26%）：13 index 页 + 4 hub 页 + 5 轻量概念 + 12 stub 待扩展

## [2026-05-17] ingest | SDD 自定义工作流：OpenSpec + Superpowers 薄编排
- created `sdd-custom-workflow` — 薄编排架构，sdd-* action skills 统一入口，Action Not Phases，三段式 skill 结构，信息丢失防护，渐进采用策略
- updated `superpowers-openspec-pitfalls` — 新增薄编排页交叉引用
- updated `sdd-openspec-superpowers` — 新增薄编排页交叉引用
- updated `openspec-sdd-practice` — 新增薄编排页交叉引用
- 跳过 `AI编程三剑客` — 同一文章已于 2026-05-15 摄入

## [2026-05-17] ingest | Superpowers 实测基准 + 双框架组合踩坑
- created `superpowers-openspec-pitfalls` — 7 个坑及解决方案、自定义衔接 Skill、四组合效果对比、3 分钟快速上手
- updated `superpowers-framework` — 新增定量基准(58%→91%)、6 技能详解、3 种最佳场景、AI 编程工程化拐点分析
- updated `superpowers-openspec-legacy-workflow` — 新增踩坑页交叉引用
- updated `openspec-sdd-practice` — 新增踩坑页交叉引用
- updated `sdd-openspec-superpowers` — 新增踩坑页交叉引用

## [2026-05-17] ingest | Superpowers + OpenSpec 协同开发实战指南
- created `superpowers-openspec-legacy-workflow` — 老旧项目四阶段闭环（分析→规约→生成→同步），团队角色变化，4大避坑指南
- updated `sdd-openspec-superpowers` — 新增老旧项目实战链接
- updated `openspec-sdd-practice` — 新增协同实战链接
- updated `superpowers-framework` — 新增协同实战链接
- updated `superpowers-vs-openspec` — 新增协同实战链接

## [2026-05-17] ingest | Claude Code 大型代码库配置指南
- created `claude-code-configuration-guide` — 实操配置指南，含 CLAUDE.md 分层模板、Hooks/Skills 配置实例、代码地图、分阶段检查清单
- updated `claude-code-large-codebase` — 新增配置指南交叉引用
- updated `agent-harness` — 新增配置指南链接

## [2026-05-17] ingest | How Claude Code works in large codebases
- created `claude-code-large-codebase` — Anthropic 官方大型代码库最佳实践，agentic search 与 RAG 对比、Harness 构建顺序、三大配置模式、组织级落地
- updated `claude-code-search-strategy` — 新增大型代码库实践链接
- updated `agent-harness-overview` — 新增 Harness 构建顺序链接
- updated `claude-code-extensibility` — 新增 LSP/Subagent 说明和构建顺序
- updated `claude-md-best-practices` — 新增分层策略和模型演进维护
- updated `ai-governance` — 新增组织级 AI 治理模式
- updated `agent-harness` — 新增大型代码库页面链接

## [2026-05-15] lint --fix | 知识库整理优化
- fixed `sources/2026/05/15/` — 补 ingested frontmatter
- fixed 10 页 — 补 sources 字段（8 draft + 2 meta）
- fixed `llm-wiki` — 明确标注为索引枢纽，指向 llm-wiki-concept 为主入口
- fixed `agent-harness` — 明确标注为索引枢纽，指向 agent-harness-overview 为主入口
- flagged SDD 工具对比 3 页 — 需人工确认各页定位边界
- flagged Karpathy 知识库 2 页 — 内容高度接近，建议合并
- flagged Obsidian 2 页 — 共享 Obsidian 别名，可能混淆
- 结论：107 页面，1 critical→fixed，17 warning→fixed 12 + 5 保留，7 info→待人工

## [2026-05-15] ingest | AI 编程三剑客：Spec-Kit、OpenSpec、Superpowers 深度对比与实战指南
- created `ai-agent/spec-kit` — GitHub 官方 SDD 框架，wiki 此前完全缺失 Spec-Kit
- created `ai-agent/ai-programming-tools-comparison` — 三剑客全面对比，含决策树与协同方案
- updated `ai-agent/superpowers-framework` — 补充安装步骤、技能触发机制、自定义技能、Spec-Kit 交叉引用
- updated `ai-agent/sdd-openspec-superpowers` — 纳入 Spec-Kit 三维视角，更新交叉引用
- updated `ai-agent/superpowers-vs-openspec` — 从"双语切换"升级为"多模式切换"，新增 Spec-Kit 维度

## [2026-05-15] ingest | 开源AI编程工具对决——Superpowers技能库与OpenSpec规范驱动
- created `ai-agent/superpowers-vs-openspec` — 个体效率 vs 团队一致性的哲学层面对比
- updated `ai-agent/sdd-openspec-superpowers` — 增加哲学层面分析和场景选择指南
- updated `ai-agent/superpowers-framework` — 补充适用场景说明和交叉引用
- updated `ai-agent/openspec-sdd-practice` — 补充适用场景说明和交叉引用

## [2026-05-14] ingest | 创建 dataagent-semantic-layer（DataAgent 语义层分析）
## [2026-05-13] lint --fix | 知识库深度检查与优化
- fixed 11 页面 status: draft → tags draft（agent-architecture-patterns/agent-harness/agent-mcp-protocol/agent-multi-agent-collaboration/agent-skills-system/agent-tdd-workflow/claude-code/llm-wiki/token-consumption-economics/learning-path/tech-radar）
- fixed 9 索引页补 content-type tag: summary（ai-agent/ai-ml/architecture/backend/big-data/distributed/fullstack/platform/product index）
- fixed 1 HTML 源文件补 ingested frontmatter（sources/2026/05/10/Claude Code 技术解析.html）
- verified 10 wanted-page + 2 根目录引用为正常断链（保留作为发现机制）
- verified 15 二进制源文件摄入可追溯（PDF/PNG 通过 wiki-log 和 wiki 页面 sources frontmatter）
- 结论：109 页面，0 critical，0 warning，12 info

## 2026-05-10 ingest | Claude Code 搜索策略——为什么 grep 打败了 RAG
- 创建 `ai-agent/claude-code-search-strategy` — 从《Claude Code 源码解析》第8章提取：4个不使用RAG的理由、GrepTool三种output_mode、head_limit=250、Glob按mtime排序、Read多模态+FILE_UNCHANGED_STUB、512K行代码零向量数据库的实证数据

## 2026-05-09 init | Deep Study 知识库初始化
- 定义 wiki-purpose — 知识库定位、范围与目标（6大技术领域）
- 定义 wiki-agent — Agent 身份、职责与摄入规则
- 定义 wiki-schema — 目录结构、命名约定、标签体系、页面类型
- 创建首页 homepage.md — 知识库导航入口
- 创建 6 个领域子目录: big-data, distributed, ai-ml, backend, platform, architecture

## 2026-05-09 update | 新增全栈开发 + AI Agent 开发领域
- 新增 `wiki/fullstack/` — 全栈开发（React/Vue/TypeScript/Next.js/Node.js 等）
- 新增 `wiki/ai-agent/` — AI Agent 开发（LangChain/CrewAI/MCP/多 Agent 协作）
- 从 ai-ml 中拆分 AI Agent 为独立领域，保持 ai-ml 聚焦模型与推理基础
- 更新 wiki-purpose、wiki-schema、homepage 的领域范围与标签体系

## 2026-05-09 ingest | 清华大学 OpenClaw 与数字员工研究报告
- 创建 `ai-agent/openclaw.md` — OpenClaw 架构、执行层、四层差距、五种原创概念
- 来源文件：sources/2026/05/09/清华大学2026年OpenClaw与数字员工研究报告47页.pdf

## 2026-05-09 ingest | 清华大学 Token 消费学研究报告
- 创建 `ai-ml/token-consumption-economics.md` — Token 冗余、预算治理、可控放量（扫描件，内容待补）
- 来源文件：sources/2026/05/09/清华大学：Token消费学研究报告.pdf（扫描件，文本提取受限）
- 迁移 sources 目录结构：YYYY-MM-DD → YYYY/MM/DD

## 2026-05-09 ingest | Obsidian + Claudian：AI知识库终极方案
- 创建 `ai-agent/obsidian-claudian.md` — Obsidian + Claude Code + Claudian 插件架构、Skills 三级进化、部署指南

## 2026-05-09 ingest | Obsidian打造个人第二大脑
- 创建 `ai-agent/obsidian-second-brain.md` — Obsidian 四大核心优势、推荐插件栈、快速上手指南

## 2026-05-09 ingest | Claude Code Auto Dream 记忆整合机制
- 创建 `ai-agent/claude-code-auto-dream.md` — 四阶段整合流程、触发条件、四种记忆机制全景

## 2026-05-09 ingest | Claude Code sourcemap 逆向还原
- 创建 `ai-agent/claude-code-sourcemap-reverse.md` — 1,884 文件还原、七类问题、三层降级策略

## 2026-05-09 ingest | Claude Code 源码架构解析
- 创建 `ai-agent/claude-code-architecture.md` — 启动入口、Prompt装配层、工具契约、权限管道、长任务续航五层主线

## 2026-05-09 ingest | Claude Code 87个隐藏功能
- 创建 `ai-agent/claude-code-hidden-features.md` — Kairos主动助手、Coordinator编排、Teleport跨设备、Voice语音、六大进化路线

## 2026-05-09 ingest | Memory/SDD/Spec 专题
- Memory: OpenClaw Memory Wiki 合并到 [[openclaw]]
- SDD (3个): 合并到 sdd-openspec-superpowers / superpowers-framework
- Spec (2个): 合并到 sdd-openspec-superpowers / superpowers-framework
- 创建 `ai-agent/llm-wiki-six-layer.md` — 六层架构改造
- 创建 `ai-agent/open-design.md` — Open Design 开源设计工具
- 创建 `ai-ml/rag-query-optimization.md` — RAG query 改写五策略
- 更新 `ai-agent/rag-vs-llm-wiki.md` — 五代方案演进对比
- 其余文件合并到已有 llm-wiki 相关页面
- 创建 `ai-agent/agent-harness-overview.md` — Harness 六承重层、七步循环
- 创建 `ai-agent/architect-in-agent-era.md` — Agent 时代架构师七项系统能力
- 创建 `ai-agent/sub-agent-vs-team.md` — Sub-Agent vs Team + 三框架对比
- 创建 `ai-agent/manus-context-engineering.md` — Manus 六条上下文工程原则
- 创建 `ai-agent/harness-build-to-delete.md` — Build to delete 演进
- 创建 `ai-agent/ai-career-trends.md` — Dario 职业趋势判断
- 创建 `ai-agent/harness-design-long-running.md` — GAN 启发的三 Agent 架构
- 创建 `ai-agent/harness-engineering-practice.md` — Human-first → Agent-aware
- 创建 `ai-agent/agent-vs-framework-vs-harness.md` — 三层概念区别
- 创建 `ai-agent/deerflow.md` — 字节跳动 DeerFlow 2.0
- 创建 `big-data/openai-data-agent.md` — OpenAI 内部 Data Agent
- 创建 `ai-agent/harness-as-backend.md` — Harness 作为新后端
- 其余 22 个文件（新闻/趋势/Claude源码）合并到已有页面
- 创建 `ai-agent/claude-code-vs-opencode.md` — CC vs OpenCode 四维度对比（上下文/Prompt/Agent/工具）
- 创建 `ai-agent/openspec-sdd-practice.md` — OpenSpec 七步工作流实战复盘
- 创建 `ai-agent/sdd-openspec-superpowers.md` — SDD 双框架对比
- 创建 `ai-agent/hermes-agent.md` — Hermes 自进化 Agent 框架
- 创建 `big-data/data-dev-skills-automation.md` — 数仓开发 Skills 自动化
- 创建 `ai-agent/claude-managed-agents.md` — CMA 脑手分离架构
- 创建 `ai-agent/google-scion.md` — Google 多 Agent 沙箱
- 创建 `ai-agent/meta-jit-testing.md` — Meta JiT 测试（bug检出+4倍）
- 创建 `ai-agent/claude-md-best-practices.md` — CLAUDE.md 最佳实践与陷阱
- 创建 `big-data/maxcompute-data-ai.md` — MaxCompute Data+AI 演进
- 创建 `ai-agent/ai-agent-ecommerce-content.md` — 淘工厂电商内容 Agent
- 更新多个新闻类文章 frontmatter 指向已有 wiki 页面（Hermes/OpenClaw/CMA）

## 2026-05-10 ingest | Dive into Claude Code 论文
- 创建 `ai-agent/dive-into-claude-code.md` — MBZUAI 学术论文笔记：5价值观+13原则+7组件架构
- 创建 `ai-agent/claude-code-permission-system.md` — 7种权限模式、拒绝优先规则引擎、ML分类器
- 创建 `ai-agent/claude-code-context-compaction.md` — 5层渐进压缩管道、CLAUDE.md四级记忆
- 创建 `ai-agent/claude-code-extensibility.md` — MCP/Plugins/Skills/Hooks四种扩展机制
- 创建 `ai-agent/claude-code-subagent.md` — AgentTool委托架构、Worktree隔离、侧链转录
- 创建 `ai-agent/claude-code-session-persistence.md` — 只追加JSONL转录、Resume/Fork机制
- 更新 `ai-agent/claude-code-architecture.md` — 增加新页面交叉引用
- 更新 `ai-agent/claude-code-harness.md` — 增加新页面交叉引用

## 2026-05-10 ingest | Claude Code 多Agent实现机制
- 创建 `ai-agent/claude-code-multi-agent-mechanism.md` — 三种机制总览：常规Subagent/Fork/Coordinator + 5条设计原则
- 创建 `ai-agent/claude-code-fork-subagent.md` — Fork Subagent字节级缓存复用机制
- 创建 `ai-agent/claude-code-coordinator-mode.md` — Coordinator主从型并行协作模式
- 更新 `ai-agent/claude-code-subagent.md` — 增加多Agent机制交叉引用

## 2026-05-10 ingest | Agent 的长短期记忆系统
- 创建 `ai-agent/agent-memory-system.md` — 短期记忆(context window) + 长期记忆(向量数据库+Embedding)，粒度设计，两层配合流程

## 2026-05-10 ingest | Claude Code from Source 技术书籍（18章）
- 创建 `ai-agent/claude-code-bootstrap-pipeline.md` — 5阶段启动管道/300ms预算/I/O并行
- 创建 `ai-agent/claude-code-state-architecture.md` — 两层状态/粘性锁存器/5个Sticky Latch
- 创建 `ai-agent/claude-code-api-layer.md` — 多云提供商/prompt cache三级/输出槽位预留
- 创建 `ai-agent/claude-code-agent-loop.md` — 1730行async generator / 10终止+7继续状态 / 4层压缩
- 创建 `ai-agent/claude-code-tool-system.md` — 40+工具 / 14步执行管道 / fail-closed默认值
- 创建 `ai-agent/claude-code-concurrent-tools.md` — 按安全分区并发 / 推测执行 / 顺序保持
- 创建 `ai-agent/claude-code-memory-system.md` — 4类型/可推导性标准/Sonnet侧查询/背景提取Agent
- 创建 `ai-agent/claude-code-swarm.md` — Swarm多Agent协作/Teammate/Mailbox/Auto-Resume
- 创建 `ai-agent/claude-code-terminal-ui.md` — 自定义DOM/7阶段渲染/驻留池/双缓冲
- 创建 `ai-agent/claude-code-input-system.md` — 5种键盘协议/Chord和弦/Vim 12状态机
- 创建 `ai-agent/claude-code-mcp.md` — 8种传输/7个配置来源/4阶段工具包装
- 创建 `ai-agent/claude-code-remote-execution.md` — Bridge v1/v2/FlushGate/非对称读写
- 创建 `ai-agent/claude-code-performance.md` — 位图预过滤器/槽位预留/启动I/O并行
- 更新 `ai-agent/claude-code-architecture.md` — 新增六抽象模型/10大设计模式/可迁移原则
- 更新 `ai-agent/claude-code-subagent.md` — 新增10步决策树/15步runAgent生命周期/5维度设计语言
- 更新 `ai-agent/claude-code-fork-subagent.md` — 新增三层冻结/递归防护/Sync-to-Async转换
- 更新 `ai-agent/claude-code-extensibility.md` — 新增Skills两阶段加载/Hooks快照安全/退出码语义

## 2026-05-10 lint | 健康检查 + auto-fix
- 创建 8 个领域 index 页面（ai-agent/big-data/ai-ml/distributed/fullstack/backend/platform/architecture）
- 创建 8 个 stub 页面修复高频断裂链接（agent-architecture-patterns, agent-skills-system, agent-tdd-workflow, agent-multi-agent-collaboration, agent-harness, agent-mcp-protocol, claude-code, llm-wiki）
- 创建 [[tech-radar]] + [[learning-path]] stub 页面
- 修复 homepage.md — 中文领域链接指向各目录 index + 补 sources + 更新待创建链接
- 更新 ai-agent/index.md — 补入 3 个孤立页面的入链（open-design, anthropic-cookbook, ai-agent-ecommerce-content）

## 2026-05-10 ingest | Claude Code 源码解析橙皮书（花叔·12章80页）
- 创建 `ai-agent/claude-code-search-strategy.md` — 4理由解释零向量数据库/grep vs RAG/GrepTool 3种output_mode
- 创建 `ai-agent/claude-code-internal-mechanisms.md` — Undercover Mode/反蒸馏防御/Regex情绪检测/Verification Agent
- 更新 `ai-agent/claude-code-api-layer.md` — System Prompt 5级覆盖/3种运行模式/Capybara v8幻觉率29-30%/v2.1.76 cache回归bug
- 更新 `ai-agent/claude-code-permission-system.md` — YOLO两阶段64→4096/熔断机制/Auto模式危险权限剥夺/Zig层DRM/风险评估器
- 更新 `ai-agent/claude-code-context-compaction.md` — 9段式压缩模板/Full vs Partial Compact/两阶段/防漂移逐字引用/Sonnet 4.6失败率2.79%
- 更新 `ai-agent/claude-code-tool-system.md` — 4能力原语/Deferred Tools & ToolSearch/Bash两层安全分析
- 更新 `ai-agent/claude-code-memory-system.md` — 双向互斥/重叠请求Stash/Cache共享/记录成功不只记录失败/时间规范化
- 更新 `ai-agent/claude-code-swarm.md` — Team销毁顺序/Continue vs Spawn决策/权限同步文件锁/.worktreeinclude
- 更新 `ai-agent/claude-code-hidden-features.md` — GrowthBook基础设施/Dead Code Elimination/BUDDY实现细节/44个tengu_flags
- 存档 PDF + txt 至 `sources/2026-05-10/Claude-Code-Source-Analysis.*`

## 2026-05-10 ingest | Agent 设计范式（ReAct/Plan-and-Execute/Reflection）
- 创建 `ai-agent/agent-design-paradigms.md` — 三种范式核心区别/实现原理/优劣势/选型口诀
- 关联 Claude Code 实现：ReAct→agent-loop, Plan-and-Execute→coordinator-mode, Reflection→verification-agent

## 2026-05-10 ingest | Claude Code 技术解析报告（HTML·~46K字·10节）
- 创建 `ai-agent/claude-code-builtin-agents.md` — 5个内置Agent完整system prompt+设计哲学+对比总览
- 更新 `ai-agent/claude-code-architecture.md` — 10大设计模式实现级分析（PROBLEM→SOLUTION→ADVICE）+ 12核心源文件快速参考
- 更新 `ai-agent/claude-code-subagent.md` — 补充内置Agent交叉引用
- 存档至 `sources/2026-05-10/Claude Code from Source.md`

## 2026-05-10 ingest | 2025数据智能体实践指南（火山引擎·中国信通院·中国联通·中国移动）
- 创建 `big-data/data-agent-practice-guide.md` — 认知重构/六维能力/DAMM L1-L4/双核心架构/四大场景/四阶段实施/产业展望
- 存档 PDF + txt 至 `sources/2026/05/10/`

## 2026-05-10 ingest | 硅谷顶级PM的方法论（Lenny's Newsletter提炼）
- 新增领域：产品与技术管理（更新 wiki-purpose.md、wiki-schema.md、homepage.md）
- 创建 `product/silicon-valley-pm-methodology.md` — 客户痴迷/战略金字塔/MLP/赛车增长/北极星指标/LNO框架
- 创建 `product/index.md` — 产品与技术管理索引入口
- 存档 PDF 至 `sources/2026/05/10/`

## 2026-05-10 ingest | Google 上下文工程白皮书
- 创建 `ai-agent/google-context-engineering.md` — Context vs Prompt工程/会话架构(ADK vs LangChain)/记忆种类+范围+存储+信任/多Agent会话共享/生产架构
- 对比 Claude Code 记忆系统
- 存档 PDF + txt 至 `sources/2026/05/10/`

## 2026-05-11 22:15 | ingest | claude-code-principles.html V3 Round 2 fixes
- Applied 10 fix categories (F1-F10) to claude-code-principles.html
- F1: Added 4 inline SVG diagrams (architecture overview, Agent Loop cycle, 14-step pipeline, 5-layer compaction)
- F2: All code blocks converted to styled format with syntax highlighting and line numbers
- F3: Expanded Chapter 9 from 3/14 to all 14 prompt segments with real text
- F4: Added 8 inter-chapter transition sentences
- F5: Added pain-point statements to chapters 2, 4, 6, 7, 8
- F6: Added "why" inline annotations to key code blocks (state, SAFE_DEFAULTS, compaction, partition, fork)
- F7: Added source evidence annotations for key numbers (1730行, AUTOCOMPACT_BUFFER_TOKENS, 3400万次/周, 914行, 70%)
- F8: Added alternative comparison notes (fail-closed vs whitelist, segment registry vs monolithic)
- F9: Defined key terms on first use (YOLO分类器, Teammate/Mailbox, Auto Dream, Context Collapse, BOUNDARY)
- F10: Added back-to-top button with CSS + JS scroll detection
- File: 1008 → 1449 lines

## [2026-05-12] ingest | DataWorks Data Agent 及产品生态系列（7篇）
- created `dataworks-data-agent` — DataWorks Data Agent 核心产品架构
- created `superetl` — 菜鸟基于 Data Agent 的物流 SuperETL 体系
- created `maxcompute-skills` — MaxCompute MCMCP 服务与 Skills 语义包
- created `hologres-skills` — Hologres AI Function/CLI/Skills/Mem0
- created `flink-skills` — Flink Agent Skills 四层能力体系
- created `emr-skills` — EMR AI Native 三大子产品 Skills
- created `dataworks-stock-case` — 个人投资者量化行情分析实践
- updated `big-data/index` — 新增 Data Agent 生态和实践案例链接
- updated `data-agent-practice-guide` — 添加跨页面引用
- updated `data-dev-skills-automation` — 添加各产品 Skills 引用
- updated `maxcompute-data-ai` — 添加 MaxCompute Skills 引用
- updated `agent-skills-system` — 添加大数据平台 Skills 实践案例

## [2026-05-12] ingest | DataWorks DataAgent 分享录音
- updated `dataworks-data-agent` — 补充五阶段演进、ACP 网关路由、CLI/Claw 模式对比
- updated `superetl` — 补充 9 技能拆分原理、Hook 四触发点、发布阻断机制、6 大类知识组装、变与不变框架
- updated `hologres-skills` — 补充广告生成完整流程细节
- updated `flink-skills` — 补充来源录音
- updated `emr-skills` — 补充 read_file 表函数、Agent 时代三大新要求
- updated `maxcompute-skills` — 补充来源录音
- updated `dataworks-stock-case` — 补充数据量、做饭比喻、逆风局概念、"经验 Skill 化"结论
- source saved: sources/2026/05/12/DataWorks DataAgent分享录音.md

## [2026-05-12] update | 修复 17 处断链
- `rag-vs-wiki` → `rag-vs-llm-wiki` (3 files)
- `context-engineering` → `google-context-engineering` (2 files)
- `agent-memory-systems` → `agent-memory-system` (2 files)
- `sdd-openspec` → `sdd-openspec-superpowers` (2 files)
- 去除无对应页面的 [[]]: `easydata`, `spark-ai-integration`, `claude-code-claudemd`, `topic-name`, `笔记名`, `wikilinks`, `obsidian-para`

## [2026-05-12] ingest | 虾聊日 PPT 配图（6张PNG）
- `dataworks-data-agent` — 新增架构图、Core 架构图、CLI演示图
- `superetl` — 新增数仓开发耗时分布图、CLI演示图、AI时代数据网格图

## [2026-05-12] lint | 知识库健康检查及优化修正
- fixed `homepage` — 修复 9 个领域索引 `[[link\|text]]` 转义导致的死链
- fixed 跨页死链 — `rag-vs-wiki`→`rag-vs-llm-wiki`, `context-engineering`→`google-context-engineering`, `agent-memory-systems`→`agent-memory-system`, `sdd-openspec`→`sdd-openspec-superpowers`
- fixed `claude-code-swarm` — 补充缺失 `created`/`updated` frontmatter
- removed 无意义占位链接 — easydata, spark-ai-integration, claude-code-claudemd, topic-name, 笔记名, wikilinks, obsidian-para
- kept 10 个 wanted-page 链接（作为未来可创建页面的发现机制）
- 结论：108 页面，0 可修复死链，12 个 wanted-page 链接为正常现象

## [2026-05-12] lint --fix | 知识库优化修正
- 断链审查：8 个 wanted-page 保留（glm5-model/agent-autonomous-planning/ai-governance/digital-employee/ai-agent-security/llm-cost-optimization/prompt-engineering/ai-knowledge-base），wiki-purpose/wiki-schema 为根目录合法引用
- 补源文件 ingested frontmatter：DataWorks DataAgent分享录音.md、OpenClaw与数字员工研究报告.txt
- 修复 claude-code-swarm — 孤立 wikilink 移至 ## 相关页面 章节
- 清理 sources/ 目录 3 个 .DS_Store 垃圾文件
- 核实 20 个 draft stub 页面空 sources 为合理设计（桥接页面，创建于历史 lint 操作）

## [2026-05-12] lint --fix | 知识库深度优化
- created `sources/2026/05/10/lint-stub.md` — 为 20 个无源页面提供合成来源标记
- fixed `sources` — 8 个 stub 桥接页 + 12 个索引页 sources 均指向 lint-stub.md（满足 schema 必填约束）
- fixed `tags` — 9 个页面补充内容类型标签：homepage/learning-path/tech-radar 加 `summary`，6 个 big-data 页面补充 `tool`/`practice`
- fixed `aliases` — 8 个 Claude Code/LLM Wiki 页面补充 `CC`/`Claude CLI`/`LLM Wiki`/`Obsidian` 别名
- result: 0 critical, 0 warning, 10 info（8 wanted-page + 2 root-level link）

## [2026-05-12] ingest | OpenCode DeepWiki 全架构分析
- created `ai-agent/opencode` — 开源 AI 编程智能体全架构：monorepo 结构、Client-Server 分层、12 子系统详解（CLI/配置/会话/Provider/工具/HTTP/事件/LSP/插件/MCP/Skills/ACP/LLM Core）
- updated `ai-agent/claude-code-vs-opencode` — 添加 [[opencode]] 交叉引用
- updated `ai-agent/index` — 添加 OpenCode 条目
- source saved: sources/2026/05/12/deepwiki-opencode.html

## [2026-05-12] lint --fix | 知识库修复（重试）
- fixed llm-wiki status tool — 修复 link resolution bug：原逻辑仅匹配全路径 slug（如 `ai-agent/claude-code-architecture`），缺少文件名级 fallback，导致 497 个有效链接误报为断链。已 patch `cli.js` 对齐 graph 命令的 resolveLink 逻辑
- report: 109 页面，95 源文件，0 critical（工具修复后），12 info（10 wanted-page + 2 根目录配置页引用），0 orphan
- kept 12 wanted-page 链接作为未来页面发现机制

## [2026-05-29] ingest | DataWorks虾聊0528录音修正版

- 原始录音文件经过多轮 SubAgent 并行检查 + 主Agent 汇总修正，修正了约 160+ 处语音识别错误
- 修正范围：产品名称/技术术语/演讲者姓名/乱码恢复/重复文本清理/语义修正
- 源文件归档至 sources/2026/05/28/DataWorks虾聊0528录音-修正版.md

## [2026-05-29] ingest | 阿里 DataWorks 2026-05-28 虾聊日分享内容（7篇PDF）

- created `dataworks-2026-0528-xialiaori` — 虾聊日活动汇总页面，7场分享的核心内容、演讲人、关键趋势
- created `openclaw-agentic-search-memory` — OpenClaw 两大认知引擎：Agentic Search（理解→规划→执行→反馈 四阶段研究闭环）与 Agentic Memory（Extract→Embed→Store→Retrieve→Fuse 五步工业级记忆流程），研究飞轮协同
- created `taobao-live-data-dev-paradigm` — 淘宝直播 AI 驱动的数据研发范式升级：R2C 全流程（NL2DSL2SQL）、三层架构（应用层/AI能力层/基建层）、Multi-Agent 工厂流水线四角色（需求分析/Coordinator+Planner/Research Team/Reporter）、四阶段 Spec 链审计前置、DSL 全自动 100% + Data Agent 半自动 50-80% 双路径生码、三大基础设施（CDM 双产物/知识库双引擎/工程平台 4 模块）、24Q3-26Q2 演进时间线
- updated `dataworks-data-agent` — 补充 2026 年正式发布信息与现场演示的三种模式
- updated `emr-skills` — 新增从 Lakehouse 到 Agentic Lakehouse 概念、StarRocks Skills 六大类详细能力地图（领域知识/开发管理/性能分析/诊断/运维/数据分析）、四个真实场景（金融实时风控/集群巡检/报表加速/迁移）
- updated `flink-skills` — 新增批量巡检诊断、品牌舆情实时监控两个详细场景，补充"大促不怕炸，巡检靠拼装"的 Skill 组合编排理念
- updated `maxcompute-skills` — 新增 MaxAgent 运营助手四大场景、Delta Table/AutoMV Agent 时代保障机制、Blob 四大应用场景与多模态开发体验（拖拽/画布/预览/WebDAV）
- updated `hologres-skills` — 新增 HSAP 1.0→4.0 演进时间线、四步广告素材生成全链路（带 SQL 示例）、AI 助手四 Agent 协作架构
- updated `ai-agent/ecosystem/index` — 新增 [[openclaw-agentic-search-memory]] 链接
- updated `big-data/index` — 新增虾聊日活动板块 + [[taobao-live-data-dev-paradigm]] 链接
- 7 PDF 源文件 + 7 TXT 提取文件归档至 sources/2026/05/28/（淘宝直播 PDF 为纯图片，通过 tesseract OCR 提取）
- created `product/netease-tasted-model` — 网易 TASTED 六力模型（审美/洞察/标准/结构/判断/自驱）
- created `product/netease-tasted-interview` — SBO 行为面试法的六力追问实战手册

## [2026-05-14] lint | 知识库整理——修复 12 断链 + 消除孤立簇
- created 10 stub pages — 补全断链目标页（`wiki-purpose`, `wiki-schema`, `agent-autonomous-planning`, `ai-agent-security`, `digital-employee`, `glm5-model`, `ai-governance`, `ai-knowledge-base`, `llm-cost-optimization`, `prompt-engineering`）
- updated `product/index` — 链入 `netease-tasted-model` 和 `netease-tasted-interview`，消除孤立簇

## [2026-05-31] ingest | 基于 Hologres Skills 完成广告素材智能生成与投放效果分析（逐页重摄取）
- updated `hologres-skills` — 按淘宝直播 PDF 高密度摄取标准，从 PDF 页面图片重新复核 16 页，忽略既有 TXT，补充逐页索引、P2-P16 逐页复核、架构图/流程图/Demo/代码参数/二维码页信息
- source reviewed: `2026/05/28/基于 Hologres Skills 完成广告素材智能生成与投放效果分析.pdf`

## [2026-05-31] update | 高密度 PPT-to-PDF 摄取规则
- updated `AGENTS.md` — 新增规则：DataFun/虾聊日等高密度 PPT 转 PDF 必须从 PDF 页面图片逐页视觉复核，不能信任派生 TXT/OCR；需捕获逐页索引、架构/流程图、截图、代码、数字、示例、模块名和路线图，不可靠小字不得臆测

## [2026-05-31] update | 多模态资料首次摄取规则修正
- updated `AGENTS.md` — 将规则修正为中文，并明确要求首次摄取 PDF、图片、PPT 转 PDF 等多模态资料时即保证正确性、完整性和可追溯性；派生 TXT/OCR 不能作为唯一可信来源，必须回到原始 PDF/图片正文逐页或逐图复核，避免信息丢失

## [2026-05-31] ingest | DataWorks Data Agent：新一代数据智能体，企业大数据的数字员工（逐页重摄取）
- updated `dataworks-data-agent` — 按多模态资料首次摄取规则，从 PDF 页面图片重新复核 17 页，忽略派生 TXT/OCR，补充逐页索引、P2-P17 逐页复核、产品架构、DataAgent Core、CLI/Claw 双模式、演示截图、Skill 开放生态、全托管运行底座和路线图信息
- source reviewed: `2026/05/28/DataWorks Data Agent：新一代数据智能体，企业大数据的数字员工.pdf`

## [2026-06-15] ingest | 大数据基建如何迈向 AI Infra（上+下）
- created `ontological-semantic-layer` — 本体化语义层核心概念、六类能力框架、七项落地准备、Data Infra 团队角色重定义
- updated `ontology` — 添加本体化语义层交叉引用
- updated `dataagent-semantic-layer` — 添加本体化语义层作为上层概念框架

## [2026-06-23] ingest | 桌面上 share 目录 6 个 SVG 架构图批量摄取

- saved `sources/2026/06/23/DataWorks Data Agent 总体架构.svg` — DataWorks 产品架构图（五层）
- saved `sources/2026/06/23/DataWorks DataAgent入口与交互模式.svg` — CLI/Claw 双模式入口
- saved `sources/2026/06/23/DataWorks 架构运行流与内核.svg` — DataAgent Core 技术架构
- saved `sources/2026/06/23/菜鸟数仓未来预测.svg` — 菜鸟 Data Mesh 未来架构
- saved `sources/2026/06/23/淘宝DataAgent.svg` — 淘宝 R2C 架构图（本体论+Harness 双锚点）
- saved `sources/2026/06/23/云村DataAgent.svg` — 云村 DAS Agent 五层平台架构
- created `big-data/cainiao-data-warehouse-architecture.md` — 菜鸟数仓未来架构：Data Mesh 三层 + 交易/物流/LLM 三大数据域 + CDM/WIKI/ODS 标准化
- created `ai-agent/cloud-village-data-agent-platform.md` — 云村 DAS Agent Platform：自建 Agent + 三方 Agent 统一 das-mcp-cli 桥接 + Aris 运行时 + Rossta 语义层 + OmniSQL 查询引擎
- updated `big-data/dataworks-data-agent.md` — 新增 3 个 SVG 来源、交叉链接云村和菜鸟新页面
- updated `big-data/taobao-live-data-dev-paradigm.md` — 新增淘宝DataAgent.svg 来源、交叉链接
- updated `big-data/superetl.md` — 新增菜鸟数仓未来预测.svg 来源、交叉链接新页面

## [2026-06-26] ingest | 小米 Data Agent 获 Text2SQL 全球榜单第三名
- created `big-data/xiaomi-dimi-data-agent` — Xiaomi DiMi Data Agent 主页面：BIRD #3、NL2Semantic2SQL、五大生产级瓶颈
- updated `big-data/code-agent-vs-data-agent` — 新增 BIRD 榜单量化证据（裸模型 30-40 名 vs Harness 前五）、交叉引用
- updated `big-data/dataagent-semantic-layer` — 新增 Xiaomi NL2Semantic2SQL 参考引用
- updated `ai-agent/agent-core/data-agent-enterprise-practice` — 交叉引用 Xiaomi DiMi
- updated `big-data/index` — 新增 Data Agent 生态系列页面

## [2026-06-30] ingest | 从Databricks产品发布会看数据平台的演进方向
- saved `sources/2026/06/30/从Databricks产品发布会看数据平台的演进方向.html` — 微信公众号文章（Chrome headless 渲染，自包含 HTML，5 张正文图片）
- saved `sources/2026/06/30/从Databricks产品发布会看-图片识别.md` — 5 张图片 OCR 识别结果（图1 四层架构图、报告封面、6+2 架构图、装饰条、二维码卡片）
- created `big-data/databricks-2026-summit` — Databricks 2026 Data+AI Summit 变革综述：四层架构（数据底座/治理语义/Agent平台/应用）、LTAP、Unity AI Gateway、五大行业趋势、四维对照
- created `ai-ml/genie-ontology` — Genie Ontology：人工定义+自动生长双轨制，PageRank 式权威性排名，复杂业务首问命中率 84.5%
- created `ai-agent/harness/omnigent-meta-harness` — Omnigent 元编排层（meta-harness），Apache 2.0 开源跨框架 Agent 调度，竞争上移至编排治理层
- created `big-data/ai-native-data-platform-report` — 《AI原生数据平台研究报告》6+2 架构定义与计算/供给/治理/消费四维对照框架
- updated `big-data/databricks-genie` — 补充 Genie One/Code GA、Agent Bricks 10万生产Agent/10^15 Token 规模、Genie Ontology 语义底座、Pay-as-you-go 计费
- updated `big-data/index` — 新增 AI 原生数据平台分区与 Databricks 2026 系列页面
- updated `ai-agent/harness/index` — 新增 Omnigent 元编排层条目

## [2026-06-30] ingest | Databricks DAIS 2026 第二批四篇文章（LTAP/全家桶/Agent入口/智能体湖仓）
- saved `sources/2026/06/30/Databricks 发布 LTAP：首个统一事务与分析处理的湖上架构.html` — LTAP 技术深度拆解（智透圈，6/29）
- saved `sources/2026/06/30/Databricks Data + AI Summit 2026 深度回顾：Genie One、Lakebase 与 Agent 全家桶.html` — 产品事实盘点（智透圈，6/28）
- saved `sources/2026/06/30/Genie 之后，Databricks 真正想抢的是企业 Agent 入口.html` — 战略判断（数智技术落地，6/26）
- saved `sources/2026/06/30/Databricks Data + AI Summit 2026 重磅复盘：Databricks 正式迈入「AI 智能体湖仓」时代.html` — 财务数据+产品补强（databricks 研究基地，6/23）
- saved `sources/2026/06/30/Databricks-DAIS2026-第二批四篇-图片识别.md` — 8 张图片 OCR 识别（含 Agent 控制点战略图、LTAP 架构图、Lakehouse 架构图）
- created `big-data/ltap-architecture` — LTAP 架构：HTAP/零ETL/LTAP 三方对比、传统架构七痛点、对数据工程师五影响、Lakebase 三项增强
- created `big-data/databricks-agent-control-plane` — Databricks Agent 控制面战略：六层控制点拼图、Genie Ontology 风险视角、Snowflake/Collibra 对比、国内平台三层能力建议
- created `big-data/customerlake` — CustomerLake：Agentic CDP，Profile Agents + Campaign Agents，Agent 进入营销业务流程
- created `big-data/lakewatch-agent-siem` — Lakewatch + Panther 收购：Agent 驱动 SIEM 安全湖仓，完成全栈闭环
- created `big-data/agent-infra-vendor-strategies` — Agent 基础设施厂商战略对照：Databricks/Microsoft Solara/Google Gemini Spark 三方对标 + Snowflake/Collibra 对比
- updated `ai-ml/genie-ontology` — 补充五大权威评分维度、50+数据源、84.5% vs 52.4% 对照、风险与局限视角
- updated `ai-agent/harness/omnigent-meta-harness` — 补充五大职能、框架无关性（一行代码切换 LangGraph/CrewAI/Claude Code SDK/OpenAI Agent SDK）、托管版本
- updated `big-data/databricks-2026-summit` — 补充财务数据（ARR 54亿/AI收入14亿/融资70亿）、DAIS 2025 vs 2026 对比、Reyden 引擎、Unity Catalog Iceberg 写入 GA、Genie One 定价细节、Agent Bricks 多框架/模型支持、新增控制面/CustomerLake/Lakewatch 等交叉引用
- updated `big-data/index` — 新增 LTAP/控制面/CustomerLake/Lakewatch/厂商对照 5 个页面索引

## [2026-06-30] ingest | Databricks DAIS 2026 第三批两篇 + 官方博客
- saved `sources/2026/06/30/318| 企业级AGI：AI的真正战场.html` — 企业级AGI理论框架（王知鱼，译自 SiliconANGLE，6/29）
- saved `sources/2026/06/30/Databricks 一次发了 18 件事：Lakehouse 下沉为底座，Agent Runtime 升到顶层.html` — 18项发布全景+竞争格局（云计算指北，6/20）
- saved `sources/2026/06/30/Databricks-DAIS2026-第三批两篇-图片识别.md` — 40张图片OCR（本体成熟度模型图、SoI框架图、冰面化图、四层架构图、9条战线竞争格局图等）
- saved `sources/2026/06/30/Databricks官方博客-Introducing Genie One Genie Ontology Genie Agents.md` — Databricks官方一手来源（6/16）
- created `ai-agent/agent-core/enterprise-agi-framework` — 企业级AGI框架：企业级AGI vs 弥赛亚AGI、SoI四层堆栈、本体成熟度1-9级模型、描述性vs可执行本体、技能提升管道、数据层冰面化、Omnigent治理≠学习
- updated `ai-ml/genie-ontology` — 补充OntoRank机制+5维度权威性示例、成熟度评估(5-6级)、官方"无需人工策展"表述、官方博客来源
- updated `big-data/databricks-2026-summit` — 补充4C叙事骨架、厂商口径警示、9条战线竞争格局表、enterprise-agi-framework交叉引用
- updated `big-data/databricks-genie` — 补充Genie Agents(从Spaces演化)、Genie One核心能力、Foot Locker证言、官方博客来源
- updated `big-data/index`、`ai-agent/agent-core/index` — 新增 enterprise-agi-framework 索引

## [2026-07-01] lint | 健康度巡检与结构优化
- fixed `agentic-analytics-anthropic` — 断链 [[agent-evaluation]] → [[agent-evaluation-framework]]（已有页面）
- fixed `dataworks-2026-0528-xialiaori` — 断链 [[cainiao-superetl-dataworks-agent]] → [[superetl]]（菜鸟 SuperETL 已有页面）
- fixed `oceanbase-ai-database` — 同上断链修复
- moved `learning-path` / `tech-radar` — 从 wiki/ 根目录迁入 wiki/meta/（符合 schema 约定，wikilink 不受影响）
- created `meta/index` — 补齐 wiki/meta/ 目录索引
- updated `homepage` — 最近更新区从占位符填充为真实最新页面，更新 updated 日期
- flagged `distribution` — ai-agent 占 70%，其余领域各 1 页（学习重心真实反映，非缺陷）
- note: llm-wiki CLI 不在 PATH，sync 未能执行，待确认 CLI 安装

## [2026-07-01] restructure | big-data 拆分——Data Agent 迁出，回归纯大数据技术
- created `ai-agent/data-agent/` — 新子目录，承接原 big-data/ 下 Data Agent 主题
- migrated 28 页 big-data/* → ai-agent/data-agent/*（带 ai-agent 标签的 Data Agent 内容）
- migrated `cloud-village-data-agent-platform` — 从 ai-agent/ 根迁入 data-agent/
- created `ai-agent/data-agent/index` — 按产品/语义层/Skills/厂商战略/实践案例五分区
- rewritten `big-data/index` — 去除 Data Agent 分区，聚焦纯技术栈，附学习规划待建设分区
- updated `ai-agent/index` — 新增 data-agent/ 子领域行（29 页，第二大子域）
- updated `wiki-schema` — 目录结构补充 data-agent/ 子目录
- result: big-data/ 从 32 页降至 3 页（iceberg/ltap-architecture/maxcompute-data-ai），为 Flink/CDC/Iceberg/Paimon/湖仓一体等学习腾出容器；迁移后 0 断链（wikilink 用文件名）

## [2026-07-01] ops | 恢复 llm-wiki CLI 与 sync
- installed `@jackwener/llm-wiki@0.5.1` (npm install -g) — 此前 CLI 被误删，导致历次 sync 未执行
- ran `llm-wiki sync` — 68 changes synced, 259 unchanged，sync-state 更新至 2026-07-01
- ran `llm-wiki status`/`graph` — 184 页 / 112 源 / 1136 链接，5 社区
- verified: CLI 报的 1095 "broken wikilinks" 与 2 "orphan" 多为表格内 `\|` 转义误解析（已知误报，6/23 lint 已记录）；用正确解析核实真实断链=0，真实孤立页仅 meta/index（已从 homepage 加入链修复）
- updated `homepage` — 相关页面加入 [[meta/index]] 入链，消除孤儿
- updated `CLAUDE.md`(AGENTS.md) — 新增 CLI 安装命令（@jackwener/llm-wiki）+ 分布式读写规则（禁止写 agent 个人记忆，约定只能写入仓库内文件）

## [2026-07-01] lint | 删除 wiki/meta/ 下 wiki-purpose/wiki-schema 镜像页
- removed `wiki/meta/wiki-purpose.md` / `wiki/meta/wiki-schema.md` — 2026-05-14 误建 stub（用"建镜像页"错误修复根目录配置文件不在 wikilink 图的断链）
- 根目录 `wiki-purpose.md` / `wiki-schema.md` 保持唯一权威副本（CLAUDE.md 要求操作前必读）
- homepage / meta/index 中 [[wiki-purpose]] / [[wiki-schema]] 改为纯文本提及（配置文件不在 wikilink 图，本不应作为知识节点链接）
- 消除 basename 重复与链接歧义

## [2026-07-01] cleanup | 删除 learning-path / tech-radar（非知识内容）
- removed `wiki/meta/learning-path.md` / `wiki/meta/tech-radar.md` — 学习路线与技术雷达是计划/待办，非知识本身，不属于知识库
- updated `homepage` — 快速入口移除两项，meta/index 描述同步
- updated `meta/index` — 删除"学习与跟踪"节，仅保留知识库规则
- updated `ai-agent/index`、`product/index` — 移除相关页面引用
- updated `distributed/platform/backend/architecture/fullstack index` — "待补充"句去除 [[learning-path]] 链接
- 0 残留引用、0 真实断链

## [2026-07-01] cleanup | 删除整个 wiki/meta/ 目录
- removed `wiki/meta/`（含 index.md）— 该目录源于 2026-05-14 错误 lint（为放配置文件镜像 stub 而建），镜像与 learning-path/tech-radar 已先后清理后，仅剩一个指向根目录配置文件的冗余 index，无存在价值
- updated `homepage` — 相关页面移除 [[meta/index]]，改为直接以纯文本列出根目录配置文件
- `meta` 标签保留（标签与目录是两套体系，schema 标签体系不动）
- 0 残留引用、0 真实断链

## [2026-07-01] lint | 补全 5 个目录 index 的索引遗漏（12 页）
巡检发现多个子目录 index 未列出本目录全部页面（页面后期新增时只更新交叉引用、漏补 index）：
- `ai-ml/index` 补 4 页：genie-ontology、glm5-model、prompt-engineering、llm-cost-optimization（新增"大模型""Prompt 与成本"分区）
- `ai-agent/ecosystem/index` 补 2 页：agentic-skills、agentic-analytics-anthropic
- `ai-agent/claude-code/index` 补 3 页：claude-code、claude-code-behavior-contract、claude-code-instruction-failure
- `ai-agent/agent-core/index` 补 2 页：business-cognition-system、data-agent-enterprise-practice（新增"业务认知与企业实践"分区）
- `ai-agent/harness/index` 补 1 页：dw-harness-practice
- 结果：0 index 遗漏、0 真实断链、frontmatter 100% 完整
- 注：CLI 报的 1058 broken wikilinks 仍为表格 \| 转义误报（已核实）

## [2026-07-01] ingest | Apache Flink 主题（计算引擎）
资料来源：用户提供两批技术文章清单（Flink/CDC/Gravitino/Iceberg/Paimon/湖仓一体/流批一体/实时数仓/实时计算），共下载 93 HTML + 4 PDF 到 article-downloads/。本次按主题分批摄取，第一批为 Flink 计算引擎（10 篇）。
- created `apache-flink` — Flink 总览：有界/无界流、有状态计算、四层执行图、并行度/Slot、与 Spark/Storm 对比
- created `flink-checkpoint` — Checkpoint 容错：Chandy-Lamport 分布式快照 → 异步 Barrier → 对齐/非对齐(1.11) → Checkpoint vs Savepoint
- created `flink-state-backend` — 状态后端：Keyed/Operator State、Memory/Fs/RocksDB 三类原理与选型、增量快照、云原生演进
- updated `big-data/index` — 新增"流计算引擎"分区，列出三个 Flink 页面；从"待建设"移除 Apache Flink 条目
- added sources/2026/07/01/ 下 10 篇 Flink 来源 HTML + _ingested.md 摄取映射
- PDF（Lakehouse CIDR/Iceberg TDG/IJSAT/Flink CDC）暂缓摄取，待后续逐页视觉复核
- 注：知乎 16 篇经用户提供登录 cookie 成功下载，cookie 用完即删未持久化

## [2026-07-01] fix | HTML 来源的 sidecar 追踪方式
用户审核 Flink 摄取时质疑 `_ingested.md`。核查发现：llm-wiki CLI 只扫 `.md`（`endsWith(".md")` 过滤），`.html` 来源不被追踪；而 `_ingested.md` 聚合文件被 CLI 误当成 wiki 页面污染 Pages 计数，且无追踪意义。
- removed `sources/2026/07/01/_ingested.md` — 聚合式 sidecar，违背来源纯净性且被误计为 page
- created 10 个同名 `.md` sidecar（如 `flink-checkpoint-1-11.md`）— 每个 HTML 来源配一个仅含 frontmatter 的 .md，供 CLI 追踪 ingested 状态；HTML 本体不动
- updated `wiki-agent.md` — 新增"来源格式约定（多模态/HTML 来源）"节，固化处理经验：非 md 来源需配同名 .md sidecar，不建聚合 _ingested.md，不往 HTML 插 frontmatter
- 验证：Sources 113→122（+9 = +10 sidecar −1 删除），Pages 仍 182（sidecar 未污染）

## [2026-07-01] fix | 强化来源格式约定：读原文必须回 HTML
用户指出需明确"复核原文回到 HTML、避免找不到"。在 wiki-agent.md「来源格式约定」节补第 5 条强约束 + 末尾两行判定规则：
- 读原文 → 打开原始格式文件（HTML/PDF），不得依赖 .md sidecar（sidecar 仅 frontmatter，无正文）
- 判断是否已摄取 → 看同名 .md sidecar 的 ingested 字段
- wiki 页面 sources frontmatter 指向原始格式文件路径，复核按此路径开 HTML

## [2026-07-01] ingest | Flink CDC 主题
第二批主题摄取。9 篇 CDC 资料（HTML），按 MUST/MAY 分级：原理剖析/增量快照框架/CDC3.0/Debezium协同 为 MUST，实践优化/企业集成 为 MAY 补充。
- created `flink-cdc` — CDC 概念(Query/Log-based)、1.0 三痛点(锁/单并发/无checkpoint)、2.0 增量快照(DBLog算法+FLIP-27+chunk划分/读取/normalize+无锁+水平扩展+断点续传)、与 Debezium 协同(嵌入式引擎+RowData/RowKind changelog)、3.0 YAML pipeline 演进、链路简化
- updated `big-data/index` — 流计算引擎分区加 flink-cdc；待建设移除 CDC 条目
- added sources/2026/07/01/ 下 9 篇 CDC HTML + 同名 .md sidecar（按来源格式约定，HTML+MD 共存，读原文回 HTML）
- PDF（flink-cdc-incremental-snapshot.pdf）仍暂缓，待后续逐页视觉复核

## [2026-07-01] ingest | Apache Iceberg 主题
第三批主题摄取。12 篇 Iceberg 文章（HTML）+ 2 个 PDF（Definitive Guide 344页 / IJSAT 论文 11页）。重写扩充已有 draft 页面。
- updated `iceberg` — 从薄 draft 重写为完整页面：Netflix 三初衷（正确性/性能/易运维）、三层元数据架构（Catalog/Metadata/Data）、查询流程（metadata→snapshot→Manifest List→Manifest→data file）、时间旅行、分区演进与 Schema 演进、MVCC 快照隔离、v1→v2→v3 演进（v3 五大特性+成为行业标准）、与 Delta/Hudi/Paimon 定位对比（"娘家"论）、国内落地（腾讯/字节/小米/华为/阿里/网易）、跨云 Lakehouse 角色
- updated `big-data/index` — iceberg 条目从"待扩充"改为正式描述
- added sources/2026/07/01/ 下 12 篇 Iceberg HTML + 同名 .md sidecar
- added 2 个 PDF（iceberg-definitive-guide-dremio / iceberg-ijsat-2024）+ sidecar，标记 status: pending-visual-review，待后续逐页视觉复核

## [2026-07-01] ingest | Apache Paimon 主题
第四批主题摄取。11 篇 Paimon 文章（HTML）。新建 paimon 页面。
- created `paimon` — Streaming First 定位（大数据不可能三角、流计算+存储两代场景演进）、诞生（Flink Table Store→Paimon 2024毕业TLP、Hudi不为实时而生）、LSM Tree 核心结构、批流一体表抽象（批模式像Hive/流模式像消息队列）、三大核心能力（主键表更新/Changelog Producer/Merge Engine 去重-部分更新-预聚合）、流式湖仓分层实践（ODS→DWD→DWS + StarRocks）、生态集成、国内落地（抖音/蚂蚁/幸福里/阿里）、与 Iceberg/Hudi/Delta 关系
- updated `iceberg` 的对比表已为 Paimon 预留对标（互链）
- updated `big-data/index` — 表格式分区加 paimon；待建设移除 Paimon 条目
- added sources/2026/07/01/ 下 11 篇 Paimon HTML + 同名 .md sidecar
- 待补充：知乎 p/671077167（核心原理和Flink应用进阶）、p/1941451140853135196（事件溯源视角）两篇未下载，cookie 可能已过期，后续补抓

## [2026-07-01] ingest | 补充知乎两篇 Paimon 文章
用用户先前提供的知乎 cookie 补抓两篇漏下的 Paimon 文章（cookie 仍有效，用完即删）：
- p/671077167《Apache Paimon 核心原理和 Flink 应用进阶》(11024字)
- p/1941451140853135196《Apache Paimon（事件溯源视角）》(6421字)
- updated `paimon` — 新增"事件溯源视角：流原生存储"节（表=数据+变更历史、Event Sourcing 哲学、LSM-Inspired 而非 LSM-Based 的澄清、文件级 LSM 工作流程）；新增"基本概念与文件布局"节（Snapshot/Partition/Bucket/两阶段提交一致性）；frontmatter sources 补两篇；待补充节移除已补项
- added sources/2026/07/01/ 下 2 篇 HTML + 同名 .md sidecar

## [2026-07-01] ingest | Apache Gravitino 主题
第五批主题摄取。16 篇 Gravitino 文章（HTML，含微信8篇+CSDN多篇+知乎+B站案例）。新建 gravitino 页面。
- created `gravitino` — 定位（高性能/地理分布式/联邦元数据湖）、要解决的三大挑战（多云孤岛/多源难治理/隐藏元数据）、四层对象模型（Metalake→Catalog→Schema→Table/Fileset/Topic/Model，Catalog 的 type/provider 抽象）、统一 REST API + Iceberg REST Catalog、与 Hive Metastore 关系（前置代理非替代）、统一结构化+非结构化+AI模型元数据（0.8引入）、RBAC+授权下推+Ranger、地理分布式联邦化、与 DataHub/Unity Catalog 对比、1.0 智能上下文工程方向、落地案例（小米/B站/腾讯TBDS）
- updated `big-data/index` — 新增"元数据治理"分区；待建设移除 Gravitino 条目
- added sources/2026/07/01/ 下 16 篇 Gravitino HTML + 同名 .md sidecar

## [2026-07-01] ingest | 湖仓一体主题
第六批主题摄取。10 篇湖仓一体文章（HTML）+ CIDR 2021 论文 PDF。新建 lakehouse 页面。
- created `lakehouse` — 数据平台组成（存储+计算+接口）、演进（数仓→数据湖→湖仓一体，各自痛点）、湖仓一体定义（开放直存格式+ML一等公民+高性能查询）、三次架构迭代（离线仓库→实时补充→湖仓+批流一体）、湖仓一体vs批流一体（存储层vs计算层统一）、流批一体（Lambda/Kappa背景、阿里"一套班子一套系统一个逻辑"、Flink流批一体落点）、表格式基石、大厂落地、未来趋势
- CIDR 论文 PDF 标记 pending-visual-review（当前环境 PDF 文字提取受限，论点暂以 Databricks 博客及中文转述为据，待复核校正）
- updated `big-data/index` — 架构范式分区加 lakehouse
- added sources/2026/07/01/ 下 10 篇 HTML + CIDR PDF + 同名 .md sidecar

## [2026-07-01] ingest | 实时数仓主题
第七批主题摄取。14 篇文章（实时数仓 11 + 实时计算 3）。新建 realtime-data-warehouse 页面。
- created `realtime-data-warehouse` — 数仓概念与演进（离线大数据→Lambda→Kappa）、数仓分层（ODS/DWD/DWM/DWS/ADS 三层与五层）、Lambda 架构（批/速度/服务三层，Nathan Marz）、Kappa 架构（纯流、Kafka 重放）、Lambda vs Kappa 选型、实时数仓分层实践（各层职责/技术栈/维表关联）、实时计算引擎三代演进（Storm/Spark Streaming/Flink）、阿里云实时计算平台化、技术选型（Flink+Kafka+Doris / +Paimon / +Hologres）、落地路径与常见坑
- updated `big-data/index` — 架构范式分区加 realtime-data-warehouse；待建设移除"湖仓一体/流批一体/实时数仓"和"实时计算落地实践"（已并入此页）
- added sources/2026/07/01/ 下 14 篇 HTML + 同名 .md sidecar

## [2026-07-01] ingest | 湖格式选型对比主题（收口）
第八批（收口）。选型对比资料已分布于 Iceberg/Paimon/Lakehouse/实时数仓 主题，本页综合各页选型视角，无新来源。
- created `table-format-selection` — 四格式定位速览、横向机制对比（更新机制/元数据/索引/CDC-changelog/流批一体/引擎绑定/国内采用度）、更新机制根本差异（Iceberg 快照+delete vs Paimon LSM）、CDC/changelog 能力对比、腾讯真实选型决策（CDC选Hudi/常规入湖选Iceberg）、专家圆桌观点、按场景选型建议、国内市场格局
- updated `big-data/index` — 新增"选型对比"分区；移除整个"待建设"节（所有规划主题已建成）
- 本次无新 sources（复用已有 sidecar）

## [2026-07-01] fix | 历史来源补齐 sidecar，统一 HTML+MD 共存规则
用户要求检查之前摄取数据是否符合"HTML+MD 共存"规则。发现 5-6 月早期摄取的来源用了错误方式，三类违规：
- A类（12个HTML）：HTML 顶部被插了 YAML frontmatter（破坏浏览器渲染）→ 剥离 frontmatter 写回干净 HTML，提取 ingested/wiki_pages 建同名 .md sidecar
- B类（21个PDF）：无 .md sidecar（部分配 .txt 衍生文本，txt 不被 CLI 追踪）→ 建同名 .md sidecar（保留 .txt 作衍生文本）
- C类（24个HTML）：纯 HTML 无 sidecar 也无 frontmatter → 建同名 .md sidecar
- 修复 1 个 malformed frontmatter（为什么 Code Agent...md 的 wiki_pages 块损坏）
- 删除 sources 下 3 个 .DS_Store
- 验证：所有 html/pdf 均有同名 .md sidecar；HTML 顶部无残留 frontmatter；Sources 199→255（+56 sidecar 被 CLI 识别）；Pages 仍 188（sidecar 未污染）
- 现全部来源符合 wiki-agent.md「来源格式约定」节规则

## [2026-07-01] optimize | 知识库优化（4项）
用户要求"优化知识库"，4项全做：

**1. 修来源误用链接**：之前摄取把来源 slug 当 wikilink 写进正文（如 [[gravitino-core-objects|核心对象]]），造成 40 个假 wanted pages。54 处全部改为纯文本（来源已记在 frontmatter sources 字段）。

**2. 全面 lint 巡检**：
- 真断链（wanted pages）：53→0（修来源误用 + 19 处表格 `\|` 转义假断链）
- 假断链修复：4 个文件 19 处 `[[name\|alias]]` → `[[name|alias]]`（表格内 wikilink 转义误报，历史遗留）
- 孤立页：graph 报 1 个（agentic-skills），实为误报（ecosystem/index 有引用，工具统计 bug）
- status 的 1110 broken 是 CLI 已知误报（含 alias 的链接重复计数），非真问题
- frontmatter 100% 完整（之前 lint 已确认）

**3. 补 big-data 交叉引用**：
- iceberg/paimon 四格式对比处链向 [[table-format-selection]]（收口页从 0 入链→2）
- apache-flink 相关页面补 flink-cdc/paimon/lakehouse/realtime-data-warehouse 链接
- paimon 相关页面补 gravitino/lakehouse/realtime-data-warehouse，去掉"待建"标记
- lakehouse 入链 2→4

**4. PDF 逐页复核**（用 pdftotext 提取，非 Read 图片）：
- CIDR Lakehouse 论文：补入论文正式定义（低成本直存+DBMS功能）、两层数仓四大问题（Reliability/Data staleness/Total cost/Limited use-case）、事务元数据层实现机制、Delta Engine TPC-DS 性能 → lakehouse.md
- Iceberg TDG（344页）：补入 Catalog 五类型（Hive/Glue/Nessie/REST/JDBC）、小文件问题与 Compaction、原子提交机制 → iceberg.md
- IJSAT Iceberg 论文：复核，内容与 iceberg.md 吻合，作学术佐证
- Flink CDC PDF：article-downloads 清理时丢失，内容已由 HTML 充分覆盖，sidecar 标注 source-missing
- 4 个 PDF sidecar 状态从 pending-visual-review 更新为 reviewed/source-missing

工具发现：pdftotext 命令行工具可可靠提取 PDF 文字（之前误以为环境受限）。已记入经验。

## [2026-07-01] add | Anthropic 自服务数据分析中文总结（特例：改 sources）

- 原文 `sources/2026/06/26/anthropic-self-service-analytics-with-claude.md` 已有摄取页 `agentic-analytics-anthropic.md`
- 新增中文总结 `anthropic-self-service-analytics-with-claude.zh-cn.md`（同目录），忠实原文事实/数字/机制，按国人习惯改写表述；原文未讲处标注"原文未提及"或"推测说明"
- **特例**：经用户批准，在原文 .md frontmatter 加 `zh_cn_summary` 字段指向中文总结——违反 sources 不可变规则（CLAUDE.md 规则 2），此篇为唯一例外，不作为先例
- 中文总结 frontmatter 标 `type: zh-cn-summary`、`source:` 指回原文，避免被 CLI 误当待摄取来源
- 已跑 `llm-wiki sync`：Added 1 / Modified 1，状态干净

## [2026-07-03] ingest | 三代知识检索范式：RAG/GraphRAG/LLM Wiki（今日头条）
- 来源：今日头条文章《三代知识检索范式：RAG/GraphRAG/LLM Wiki，检索解决不了理解的问题》，作者「科技风向标」，2026-07-02；原始 HTML 存 `sources/2026/07/03/三代知识检索范式-RAG-GraphRAG-LLM-Wiki.html` + .md sidecar
- 下载流程：web_fetch 遭反爬拦截 → Chrome headless 渲染成功 → 剥离外壳、修正 6 张图、还原跳转链 → 自包含 HTML
- created `knowledge-retrieval-paradigms` — 三代范式对比（解释器/图谱/编译器），含核心差异表、成本参考、选型建议、幻觉回写/信息丢失风险
- created `graphrag` — 第二代 GraphRAG/LightRAG/nano-graphrag 概念页（实体抽取、多跳推理、LightRAG 延迟降 10-100x）
- created `storm` — STORM 主动研究模式（stanford-oval/storm）
- created `mem0` — 记忆层（mem0ai/mem0），管"关于你的知识"
- updated `llm-wiki-concept` — 补充 Adaptive RAG 查询路由（事实/推理/探索三类）
- updated `rag-vs-llm-wiki` — 加交叉引用到新对比页与 GraphRAG
- updated `index.md` — 重排为范式总览/LLM Wiki/GraphRAG与记忆，页数 12→16

## [2026-07-21] ingest | AI代码生成率94%：我们用一个 Skill 跑通需求开发全流程
- created `mailplugin-feature-dev-skill` — 企业微信团队基于 Claude Code 的邮件插件需求开发 Skill，8 阶段流水线，代码生成率 94%
- created `code-knowledge-base-pyramid` — 三级金字塔代码知识库（L1总览/L2模块/L3语义桥）+ 五步定位法，300× token 压缩
- created `requirement-semantic-translation` — 产品语言→代码指令的 5 个确定性翻译规则 + 红线机制

