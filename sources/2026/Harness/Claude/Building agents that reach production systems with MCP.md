---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-harness]
---

# Building agents that reach production systems with MCP

**文章信息**：
- **原文标题**：Building agents that reach production systems with MCP
- **来源**：https://claude.com/blog/building-agents-that-reach-production-systems-with-mcp
- **作者**：Anthropic（含多位贡献者）
- **发表时间**：April 22, 2026

---

**一段话总结**：

Agent 能力的上限取决于它能触达的系统边界。文章梳理了 Agent 连接外部系统的三条路径——直接 API 调用、CLI、MCP——并指出生产级 Agent 日益向云端迁移，MCP 凭借标准化 auth、富语义和跨客户端可达性成为关键集成层。MCP SDK 月下载量已超 3 亿，Anthropic 基于此发布了构建高质量 MCP Server 的核心模式（工具设计、代码编排、交互语义、标准 auth）、使 MCP Client 更节省 context 的技术（Tool Search 减少 85%+ 工具定义 token、程序化工具调用减少 37% token），以及将 MCP Server 与 Skills 结合打造域内专家 Agent 的方法。

---

**作者判断**：

作者明确推荐：**生产 Agent 应构建 MCP Server，且要尽可能做好**。原话："if your goal is to have production agents in the cloud reach your system, build an MCP server and make it excellent using the patterns above." 同时强调 MCP 具有"复利效应"——同一个远程 Server 随协议扩展不断获得新能力，无需重新发布。作者认为成熟集成方案会同时包含三条路径（API + CLI + MCP），但 MCP 是其中的关键层。

---

**核心内容**：

**三条连接路径对比**

- **直接 API 调用**：代码沙箱内发 HTTP 请求，适合单 Agent 对单服务。规模扩大后面临 M×N 集成问题，每对 Agent-服务都是定制集成。
- **CLI**：Agent 在 shell 中运行命令行工具，轻量快速，适合本地或沙箱容器环境。无法覆盖移动端、Web 或云端平台，auth 依赖磁盘上的凭证文件。
- **MCP**：以协议为公共层，标准化 auth、工具发现和富语义。一个远程 Server 可覆盖所有兼容客户端（Claude、ChatGPT、Cursor、VS Code 等），任意部署环境均可达。

**MCP Server 构建模式**

1. **构建远程 Server**：唯一能覆盖 Web、移动端和云端 Agent 的部署方式，是分发的前提。

2. **按意图分组工具，而非按 endpoint 映射**：少量描述清晰的工具优于 API 的一比一镜像。示例：`create_issue_from_thread` 优于 `get_thread` + `parse_messages` + `create_issue` + `link_attachment` 四步拼接。

3. **接口面宽时使用代码编排**：对 Cloudflare、AWS、Kubernetes 等需数百种操作的服务，暴露一个接受代码的薄工具层：Agent 写短脚本 → Server 在沙箱中运行 → 返回结果。参考：Cloudflare MCP Server 用 2 个工具（search + execute）覆盖约 2500 个 endpoint，仅消耗约 1K tokens。

4. **MCP Apps 与 Elicitation 增强交互语义**：
   - **MCP Apps**：工具可返回内联渲染的交互界面（图表、表单、仪表盘），采用此扩展的 Server 在 Claude.ai 等主流客户端中留存率显著更高。
   - **Elicitation**：Server 在工具调用中途暂停征询用户输入。Form mode 发送 schema，客户端渲染原生表单（用于缺失参数、确认破坏性操作）；URL mode 将用户导向浏览器（用于下游 OAuth、支付等不应经过 MCP 客户端的凭证）。

5. **使用标准化 Auth**：最新 MCP spec 支持 CIMD（Client ID Metadata Documents）完成 OAuth 客户端注册，减少首次认证摩擦和意外重新认证。Claude Managed Agents 的 Vaults 功能可持久存储 OAuth token，按会话注入并自动刷新，无需自建密钥存储。

**MCP Client 的 Context 效率优化**

- **Tool Search（按需加载工具定义）**：运行时搜索工具目录按需拉取，而非启动时全量加载到 context。测试中减少工具定义 token **85%+**，工具选择精度基本不变。

![工具搜索降低 context 使用量对比图](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/69e920e636fbec575e46319c_context-usage.webp)

- **程序化工具调用（Programmatic Tool Calling）**：在代码执行沙箱内处理工具结果（循环、过滤、聚合），只将最终输出返回给模型。测试中复杂多步工作流减少约 **37%** token 消耗。

两种模式可跨多个 Server 叠加使用：context 更精简、往返更少、响应更快。

**Skills 与 MCP 的配合**

MCP 提供工具和数据访问能力，Skills 教 Agent **如何**使用这些工具完成真实工作，二者互补。两种组合模式：

1. **打包为 Plugin**：将 Skills、MCP Server、Hooks、LSP Server、子 Agent 打包为一个可分发单元。示例：Claude Cowork 的 Data Plugin 包含 10 个 Skills + 8 个 MCP Server（Snowflake、Databricks、BigQuery、Hex 等）。

![Skills 与 MCP 协作架构示意图](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/6945b3dfa8f134d0104e4e23_How%20Skills%20and%20MCP%20work%20together%20-%20v3B%402x%20(2).png)

2. **从 MCP Server 分发 Skills**：提供商同时发布 Server 和配套 Skill，Agent 同时获得原始能力和使用 playbook。Canva、Notion、Sentry 已在 Claude connector 目录中采用此模式。MCP 社区正在开发直接从 Server 交付 Skills 的扩展，预计广泛采用后客户端将自动继承与 API 版本同步的使用知识。
