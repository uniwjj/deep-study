# Change Log

Append-only record of wiki operations. Format: `[date] verb | subject`

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

## [2026-05-14] ingest | 网易人才观 & 面试方法
- created `product/netease-tasted-model` — 网易 TASTED 六力模型（审美/洞察/标准/结构/判断/自驱）
- created `product/netease-tasted-interview` — SBO 行为面试法的六力追问实战手册

## [2026-05-14] lint | 知识库整理——修复 12 断链 + 消除孤立簇
- created 10 stub pages — 补全断链目标页（`wiki-purpose`, `wiki-schema`, `agent-autonomous-planning`, `ai-agent-security`, `digital-employee`, `glm5-model`, `ai-governance`, `ai-knowledge-base`, `llm-cost-optimization`, `prompt-engineering`）
- updated `product/index` — 链入 `netease-tasted-model` 和 `netease-tasted-interview`，消除孤立簇

