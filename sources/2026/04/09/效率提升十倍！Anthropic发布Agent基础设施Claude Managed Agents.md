---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-managed-agents]
---

## 效率提升十倍！Anthropic发布Agent基础设施Claude Managed Agents

**文章信息**：
- **原文标题**：效率提升十倍！Anthropic发布Agent基础设施Claude Managed Agents
- **来源**：https://www.toutiao.com/article/7626638874445398555
- **作者**：AI普瑞斯
- **发表时间**：2026-04-09 14:14

---

**一段话总结**：

Anthropic 推出 Claude Managed Agents 公开测试，将 Agent 上线时间从数月压缩到数天。其核心架构是"脑手分离"——把 Agent 虚拟化为 Session（会话日志）、Harness（编排引擎）、Sandbox（沙箱）三个解耦组件。解耦后延迟大幅下降，沙箱崩溃可自动恢复，安全边界更清晰（凭证永远不出现在沙箱中）。上下文管理方面，会话日志作为 Claude 上下文窗口之外的持久化记忆，通过 getEvents() 灵活查询。这一设计也支持多 Agent 并行协调。

---

**作者判断**：

作者立场客观报道，判断方向明确：Anthropic 要把 Agent 的基础设施生意握在自己手里。作者指出 Managed Agents 目前还在公开测试阶段，多 Agent 协调等高级功能仍处于"研究预览"状态，实际大规模部署的效果需要更多企业案例验证，但"方向已经很明确了"。

---

**核心内容**：

**核心问题**：从 Agent 原型到生产级，需要搞定沙箱执行、状态管理、凭证管理、权限控制、全链路追踪，可能耗费数月。Managed Agents 把这些基础设施全接过去了，开发者只需定义任务、工具和防护规则。

**架构设计——脑手分离**：

底层架构把 Agent 虚拟化为三个解耦组件：

- **Session（会话）**：只增不减的事件日志，记录 Agent 运行中的一切，独立于 Agent 运行时存在，持久化存储
- **Harness（编排引擎）**：调用 Claude 模型、路由工具调用的循环，即"大脑"
- **Sandbox（沙箱）**：Claude 执行代码、编辑文件的隔离环境，即"手"

![Managed Agents 系统架构图，Harness 为核心连接 Session、Tools、Sandbox 和 Orchestration](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/5eb6c87839864943939d71c7a2e84462~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669398&x-signature=dTj1fNDuKa559s9j69scxzDbjEY%3D)

**设计哲学**：接口保持稳定，底层实现可以自由替换——类似操作系统将硬件抽象为"进程"和"文件"，read() 不关心底层是磁盘还是 SSD。

![组件接口伪代码表格，展示 Session、Harness、Sandbox 等组件的接口定义](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/31d41bc598504d03bde2574e12dd577f~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669398&x-signature=QztG7Pf3l%2BqKRYIK%2BXXwTiKwwDA%3D)

**解耦收益**：
1. **延迟下降**：推理可在编排层拉取事件后立即开始，沙箱只在需要时按需启动
2. **可恢复性**：沙箱挂了 → 编排引擎把失败作为工具调用错误传回 Claude，Claude 决定是否重试；编排引擎挂了 → 新引擎通过 wake(sessionId) 从会话日志恢复
3. **安全隔离**：凭证永远不出现在沙箱中，Git token 在初始化时绑定到本地 remote，MCP OAuth token 存储在外部安全保险库

![Session 和 Harness 之间通过 Events 和 getEvents 进行事件传递的交互过程](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/6ced345ec4fd48f3b5fb7f5404f808f2~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669398&x-signature=ym70Ns0neTIptHkaQ9OKOIBJRsc%3D)

**上下文管理**：不压缩或裁剪上下文（传统做法不可逆），而是将会话日志作为持久化记忆。编排引擎通过 getEvents() 查询事件流的任意片段——从上次读到的地方继续、回溯、或重读关键操作前后上下文。这直接回应了此前 Agent 工具频繁压缩上下文导致 prompt 缓存命中率低的痛点。

**多 Agent 基础**：每只"手"对编排引擎只是一个统一接口 execute(name, input) → string，手不绑定在特定的脑上，脑之间可以互相传递手。

![Harness 和 Sandbox 之间的交互架构图](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/1e8a66e6aea247eeb91e5029ac4f3399~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669398&x-signature=yy2J2uq2mvAwK%2F%2FCMqL%2BZie7sWk%3D)

**已知限制**：多 Agent 协调等高级功能仍处于"研究预览"状态，需申请才能使用。

