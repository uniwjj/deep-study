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
