---
ingested: 2026-05-09
wiki_pages: [ai-agent/claw-code]
---

## 2小时破10万星！claw-code：史上最快达成GitHub里程碑的AI编码工具

**文章信息**：
- **原文标题**：2小时破10万星！claw-code：史上最快达成GitHub里程碑的AI编码工具
- **来源**：https://www.toutiao.com/article/7623977594345620008
- **作者**：doup智能AI
- **发表时间**：2026-04-02 10:07

---

**一段话总结**：

claw-code 于 2026 年 3 月 31 日在 GitHub 上线，2 小时内突破 10 万星，截至发稿已达 12.3 万星、10.2 万 Fork，成为 GitHub 历史上增长最快的仓库之一。这是一个 AI 编码工具的 Harness 系统，用 Rust 语言重写（Rust 92.9% + Python 7.1%），专注工具编排、任务自动化和 Agent 运行时上下文管理。作者是被华尔街日报报道的"超级 AI 用户"，去年使用了 250 亿个 AI 编码 Token，项目起源于一次代码泄露事件，48 小时内完成 Python 重写后又用 Rust 重构。

---

**作者判断**：

作者认为 claw-code 的成功证明 AI 编码工具的生态系统正在爆发，不只是大模型，工具层也在快速迭代。"10 万星 2 小时，这不是偶然，是市场刚需的体现。" 作者对 AI 编码工具的未来持积极态度。

---

**核心内容**：

**项目定位**：claw-code 是 AI 编码工具的 Harness 系机，核心能力包括工具编排与连接、任务自动化、Agent 运行时上下文管理。一句话概括：让 AI 编码工具变得更强大、更好用。

**技术架构（6 个核心模块）**：
- **crates/api-client**：API 客户端，支持 OAuth 和流式传输
- **crates/runtime**：会话状态与 MCP 编排
- **crates/tools**：工具定义与执行
- **crates/commands**：斜杠命令与技能发现
- **crates/plugins**：插件系统
- **crates/claw-cli**：交互式 CLI

**项目数据**：⭐ 12.3 万+ · Fork 10.2 万+ · Rust 92.9% + Python 7.1% · 贡献者 4 人

**项目起源**：起源于一次代码泄露事件，作者在 48 小时内完成 Python 重写，后又用 Rust 重构追求极致性能。作者是华尔街日报报道的"超级 AI 用户"，去年使用了 250 亿个 AI 编码 Token。

**行业背景**：2026 年 AI 编码工具正从"能用"走向"好用"，claw-code 这类 Harness 工具正好切中痛点。

**⚠️ 注意事项（来自评论区）**：据评论区反馈，Rust 重写版并非完整复刻 TypeScript 版的功能。目前缺失的功能包括：插件加载器/安装/更新流程、Hook 执行（配置解析了但不执行）、远程 Agent（Task/Team 相关功能缺失）、MCP 深度集成（只有 stdio/bootstrap）、Skills 注册（没有打包 pipeline）。PARITY.md 明确指出："It is not feature-parity with the TypeScript CLI."


