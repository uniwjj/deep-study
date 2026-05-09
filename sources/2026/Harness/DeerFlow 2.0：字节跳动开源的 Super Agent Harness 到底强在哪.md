---
ingested: 2026-05-09
wiki_pages: [ai-agent/deerflow]
---
## DeerFlow 2.0

**文章信息**
- **原文标题**：DeerFlow 2.0：字节跳动开源的 Super Agent Harness 到底强在哪？
- **来源**：<https://www.toutiao.com/article/7622032526114849334>
- **作者**：编程进阶社
- **发表时间**：2026-03-28 04:19

**一段话总结**：字节跳动于 2026 年 2 月 28 日在 GitHub 发布 DeerFlow 2.0（Deep Exploration and Efficient Research Flow），一上线即冲上 Trending 第一。这是一个完全重写的版本（与 v1 无共享代码），核心定位不是 Agent 框架而是 **Harness（马具）**——一个 Super Agent 的"组装厂"，负责把 Sub-agents、Memory、Skills、Sandbox 等组件拼装协同。架构上用 LangGraph 做 Agent 编排、支持 Claude Code 作为推理引擎（`supports_thinking: true`）、强制沙箱执行、集成字节自有搜索工具 InfoQuest，支持 Doubao-Seed-2.0-Code、DeepSeek v3.2、Kimi 2.5、GPT-4、Claude Sonnet 4.6 等模型。[编者注：有实际用户评论反馈"比较卡、Sandbox 与宿主机共享文件做得很烂"，评价不如阿里的 Hiclaw。]

---

**核心事件**：字节跳动开源 DeerFlow 2.0，定位为 Agent "组装厂"而非自建一切的框架，用 LangGraph 做编排、Claude Code 做推理、强制沙箱执行，主打可组合架构。

### DeerFlow 是什么？

官方定义：**Deep Exploration and Efficient Research Flow**（深度探索和研究工作流框架）。

更准确的描述：它是 Super Agent 的**组装厂**。与大多数从头实现一切的 Agent 项目不同，DeerFlow 更像一个**调度中心**，把现成组件拼起来协同工作：

```
DeerFlow
├── Skills & Tools（技能系统）
├── Sub-Agents（子代理）
├── Sandbox（沙箱）
├── Memory（记忆系统）
└── Claude Code Integration（Claude Code 集成）
```

这种思路像"乐高"——不自己造积木，提供标准接口让各种积木能插在一起。

### 核心架构

2.0 是完全重写版本，核心流程：

1. **理解任务**：规划器拆解任务
2. **调度子代理**：SubAgentPool 执行计划
3. **沙箱中执行**：SandboxManager 运行代码
4. **记住上下文**：MemorySystem 保存状态

关键技术选择：**用 LangGraph 做 Agent 编排**，不重复造轮子，直接站在 LangGraph 肩膀上。支持 Claude Code 的推理模型（`supports_thinking: true`）。

### 与其他 Agent 的对比

| 特性 | DeerFlow | OpenHands | AutoGPT | Claude Code |
|------|---------|-----------|---------|-------------|
| Sub-agents | ✅ | ❌ | ✅ | ✅ |
| Memory | ✅ | 有限 | ✅ | 有限 |
| Sandbox | ✅ | ✅ | ❌ | ❌ |
| Skills | ✅ | ✅ | ❌ | ✅ |
| Claude Code 集成 | ✅ | ❌ | ❌ | N/A |
| LangGraph | ✅ | ❌ | ❌ | ❌ |

三个关键差异点：

**1. Sandbox 是标配**：强制使用沙箱执行，不安全行为（文件操作、网络请求）都在隔离环境里跑——之前有 Agent 删库跑路的教训。

**2. Claude Code 可直接接入**：配置 Claude Sonnet 4.6 作为推理引擎，支持 Thinking 模式。

**3. InfoQuest 集成**：接入字节自己的智能搜索和爬取工具，意味着 DeerFlow 不仅能执行代码，还能**自己去网上搜资料**。

### 部署方式

最简单的是 Docker：

```bash
git clone https://github.com/bytedance/deer-flow.git
cd deer-flow
make config          # 生成配置文件
# 编辑 config.yaml 填入 API Key
make docker-start    # 访问 http://localhost:2026
```

本地开发：`make install && make dev`。

### 适合谁？

- **想自己组装 Agent 的人**：不想要黑盒，要可控可定制
- **研究 Agent 架构的人**：代码开源，可直接看调度逻辑
- **企业级应用**：Sandbox + Memory + Skills 组合适合自动化工作流

如果只是找个 Agent 帮写代码，Claude Code 或 Cursor 更合适。但想**理解 Agent 底层逻辑**或**构建自己的 Agent 系统**，DeerFlow 是很好的参考。

### 总结

DeerFlow 2.0 最大的价值不是自己有多强，而是提供了一个**可组合的架构**：用 LangGraph 做编排、Claude Code 做推理、Sandbox 做安全隔离、Memory 做长期记忆、Skills 做能力扩展。这种"组装思维"可能是未来 Agent 发展的方向——**与其自己造轮子，不如做一个轮子商店**。

**参考链接**：
- GitHub: https://github.com/bytedance/deer-flow
- 官网: https://deerflow.tech

