---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-harness]
---

## 开源Agent架构的设计与实现之：OpenCode

**文章信息**：
- **原文标题**：开源Agent架构的设计与实现之：OpenCode
- **来源**：https://www.agent-io.com/posts/Agent-Analysis_opencode
- **作者**：未知
- **发表时间**：未知

---

**一段话总结**：

本文基于 OpenCode v0.14.1 深度分析了这个专为终端设计的开源 AI 编码 Agent 框架。OpenCode 采用 TypeScript 服务端 + 多客户端（TUI/CLI/IDE）的 C/S 架构，核心通过 ReAct 主循环（推理→行动→观察）驱动 Agent 执行。系统设计包含五大核心理念：C/S 架构实现前后端分离、ReAct 循环驱动推理与行动、事件总线实现模块间松耦合通信、Session 作为上下文管理单元、以及声明式 Agent 配置模板。工具体系是 Agent 能力的边界，内置 read/write/edit/bash/grep 等 11 种工具，支持插件扩展和 MCP 协议。实测中，使用 Grok 模型的 OpenCode 能自主完成微博推送系统的设计、编码与验证，虽流畅度不及 Claude Code，但整体表现令人满意。作者认为 OpenCode 设计理念先进，尤其在工具体系和存储体系上对泛化 Agent 系统设计有参考价值，但仍存在缺乏沙箱隔离、JSON 存储性能瓶颈、TUI 中文显示等问题。

---

**作者判断**：

作者明确肯定 OpenCode 的设计理念和架构参考价值，原文："OpenCode 是一个设计理念先进但仍在打磨细节的工具"。作者认为其最大优势在于多 LLM 无缝切换、100% 开源、会话持久化+可分享、事件驱动解耦、工具可扩展，以及"完备的 Agent 解决方案"对设计泛化 Agent 系统的现实参考意义。同时作者也坦诚指出五个不足：缺乏显式沙箱机制、多模态支持不完备、代码可读性不足（SessionPrompt 命名不当、ReAct 循环过于冗长）、JSON 存储性能瓶颈（上万条数据时缺乏索引）、TUI 中文显示错乱。作者整体态度是"认可设计、正视不足"。

---

**核心内容**：

**应用场景**：文章列举了四个典型场景——深度重构遗留代码（10万行 Java 项目）、多语言项目一致性维护（React/Java/Flutter 三端同步 API 变更）、调试线上问题（SSH 到服务器终端环境分析日志）、开源项目贡献（理解架构+写代码+生成测试文档）。

**五大核心设计理念**：

1. **C/S 架构模式**：TypeScript 服务端 + 多客户端（TUI/CLI/IDE 插件）。优势：前端多样化、状态管理集中、远程协作友好。

![OpenCode C/S 架构图](https://www.agent-io.com/assets/img/202510/20251003_1.png)

2. **ReAct 主循环**：推理+行动+观察循环。Tool 抽象让 Agent 能力边界清晰可控，通过配置工具集即可定制 Agent 行为，工具可独立测试、复用和组合。

3. **事件驱动架构**：模块间通过 Bus 事件总线通信，实现松耦合、可观测、实时同步（客户端通过 SSE 订阅事件流）。

4. **会话即上下文**：Session 独立管理消息历史、工具调用记录和文件状态，支持父子会话用于任务分解，会话可分享、回滚、压缩。

5. **灵活可定制的 Agent 配置**：Agent 本质是配置模板（工具权限 + 执行权限 + 系统提示 + 模型配置），内置 build（完整权限）、general（搜索优化）、plan（只能分析不能修改）三个 Agent。

**整体架构**（四层）：

![OpenCode 整体架构图](https://www.agent-io.com/assets/img/202510/20251003_2.png)

- **客户端层**：CLI、TUI（Go 实现）、IDE Ext、SDK（Go + TypeScript）
- **服务端层**：Server（Hono 框架，REST API + SSE）、Agent 模块、Session 模块、Tool 模块、Event Bus
- **基础设施层**：Storage（JSON 持久化）、LLM Provider（OpenAI/Anthropic/Google/Amazon/OpenAI 兼容）、Instance Context（运行时上下文隔离）

**端到端调用链**：Client → SDK → REST API → SessionPrompt → Tools + Messages 上下文准备 → LLM (streamText) → Tool.execute() → Event Bus → SSE → Client

**ReAct 循环核心流程**：上下文准备（系统提示+可用工具集+模型）→ 主循环（推理：构建上下文调用 LLM → 行动：执行工具 → 观察：拼接结果判断是否完成）。循环退出条件：LLM 完成推理（finishReason ≠ "tool-calls"）+ 无权限阻塞 + 无错误 + 队列为空。

**工具系统**：内置 11 种工具——read、write、edit、patch、bash、grep、glob、ls、webfetch、todo、task。扩展机制包括插件系统（@opencode-ai/plugin）、本地工具目录（项目下 tool/ 文件夹自动加载）、MCP 集成。工具注册与执行流程：ToolRegistry.register() → Agent 配置启用工具 → SessionPrompt 构建 tools 对象 → LLM 返回工具调用请求 → 权限检查 → 执行 → 结果返回 LLM。

**事件总线**：采用发布-订阅模式，使用 Instance.state() 实现多项目隔离，异步解耦发布者/订阅者，SSE 实时推送。核心事件类型包括 session.updated/deleted/error、message.updated/removed、message.part.updated/removed、permission.updated/replied。

![OpenCode 事件总线架构](https://www.agent-io.com/assets/img/202510/20251003_3.png)

**存储物理结构**：
```
~/.local/share/opencode/storage/
├── session/{projectID}/{sessionID}.json
├── message/{sessionID}/{messageID}.json
├── part/{messageID}/{partID}.json
├── project/{projectID}.json
└── share/{sessionID}.json
```

**实践测试**：任务为设计并实现微博推送系统（Java）。OpenCode 先产出设计文档，再按模块逐步构建，最终验证。过程中 Agent 自主做了两处设计调整（MySQL→H2 内存数据库、Redis→内存实现）以加速运行。人工干预仅两处：Java 11 降级到 Java 8、通过继续命令触发连续执行。

**优点总结**：多 LLM 无缝切换、100% 开源、会话持久化+可分享、事件驱动解耦、工具可扩展、完备的 Agent 解决方案。

**不足总结**：缺乏显式沙箱机制（进程级隔离缺失）、多模态支持不完备、代码可读性不足（SessionPrompt 命名不当、ReAct 循环冗长）、JSON 存储缺乏索引机制（上万条数据时性能瓶颈）、TUI 中文混排显示错乱。
