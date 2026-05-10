# Change Log

Append-only record of wiki operations. Format: `[date] verb | subject`

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
- 存档至 `sources/2026-05-10/Claude Code from Source.md`
