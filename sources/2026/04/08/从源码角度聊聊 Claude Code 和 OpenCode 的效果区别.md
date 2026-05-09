---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-vs-opencode]
---

## 从源码角度聊聊 Claude Code 和 OpenCode 的效果区别

**文章信息**：
- **原文标题**：从源码角度聊聊 Claude Code 和 OpenCode 的效果区别
- **来源**：https://www.toutiao.com/article/7626358946588033555
- **作者**：VibeCoder
- **发表时间**：2026-04-08 20:08

---

**一段话总结**：

同一个 Claude 模型在不同 Agent 框架上效果差异显著，根源在于工程投入而非模型能力。Claude Code 在上下文管理、提示词工程、Agent 通信、工具编排四个维度上的工程精细度远超 OpenCode。具体体现在：三层递进上下文压缩（Micro Compact → Session Memory → Full Compact）、静态/动态提示词分割实现 prompt cache 复用、三种 Agent 隔离级别、工具自动并行分区。OpenCode 技术选型更现代（Go + TypeScript + Zig），支持 75+ 模型提供商，但在 Agent 框架深度优化上与 Anthropic 产品有明显差距。

![Claude Code 上下文管理三层压缩机制](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/82c8d50d4db94276bdadd78899299d79~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667694&x-signature=LhjaWNwyD7ZrWg1pElcmgpXqY88%3D)

---

**作者判断**：

作者认为 Agent 框架是模型能力的放大器，"基础设施越精细，模型能力释放得越充分"。Claude Code 的工程投入集中在"看不见的基础设施"上，不体现在功能列表里但直接决定用户体验。同时作者也肯定 OpenCode 的价值：75+ 模型支持提供灵活性、完整 TUI 界面体验更好、开源社区 120K+ star 认可度高。对成本敏感、需要多模型切换或本地部署的场景，OpenCode 更合理。作者承认两者差距"不是模型能力的差距，是工程投入的差距"。

---

**核心内容**：

**一、上下文管理（差距最大的维度）**

Claude Code 三层递进压缩：
- **Micro Compact**（零 API 调用）：维护 COMPACTABLE_TOOLS 集合，纯靠 token 估算裁剪旧数据（FileRead、Bash、Grep 等工具输出），开销为零
- **Session Memory Compact**：从对话历史提取结构化"会话记忆"（决策、发现、编辑中文件），保留最近 10K-40K tokens 原始消息，更早部分压缩为摘要，配置可远程动态调整
- **Full Compact**：Fork 出专用 agent 生成完整对话摘要，真正消耗 API 调用

![Claude Code 三层压缩触发机制](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/6b1f1b2599a54486bd077d41f657b20b~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667694&x-signature=3DWzB6jcHuA3Hpi0JhvUUDZVFiE%3D)

触发机制：autoCompact.ts 中有 8K 压缩缓冲区、20K 警告阈值、3K 手动压缩缓冲区，还有连续失败 3 次的熔断器。代码注释记录：2026 年 3 月发现 1279 个 session 连续失败 50 次以上，每天浪费 25 万次 API 调用。

**OpenCode 只有一层**：overflow.ts 全部 22 行代码，token 超阈值就触发，保留最近 40K tokens 工具调用，用 LLM 生成固定模板（Goal/Instructions/Discoveries/Accomplished/Relevant files）摘要，每次都调模型。

**二、提示词管理**

Claude Code 在 system prompt 埋入 `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` 标记，切分静态/动态内容。静态部分（约 5K tokens）走跨用户 prompt cache；动态部分包含环境信息、CLAUDE.md、MCP 工具列表。Fork agent 复用父进程已渲染的系统提示字节缓冲区，共享 prompt cache。

![Claude Code 提示词静态/动态分割机制](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/2d2c6f14afea402f91152eb5cba46f86~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667694&x-signature=ACMzY2xkmarkpJaf2kSLGnMi%2BgI%3D)

CLAUDE.md 四层加载：managed（企业管理员）→ user → project（项目根目录）→ local（当前目录），支持 @include 指令。

OpenCode 根据模型 ID 字符串匹配分发 6 套 prompt 模板（anthropic.txt、gpt.txt、gemini.txt 等），无 prompt cache 优化。

**三、Agent 通信**

Claude Code 三种 Agent 类型：
- **SubAgent**：完全隔离上下文，独立工具池
- **Fork**：继承父进程完整对话历史和系统提示，共享 prompt cache，`model: 'inherit'`，权限请求冒泡到父终端，启动成本几乎为零
- **Teammate**：独立进程，通过 `~/.claude/mailbox/` 文件系统邮箱通信，支持 tmux/iTerm2/in-process 三种后端

![Claude Code 三种 Agent 通信模式对比](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/62bc6ee1bc7b4bb2b1dd6012c830156d~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667694&x-signature=nDIdLXPBxis%2FusSE5DUBo9ukEJQ%3D)

OpenCode 只有 Task Tool 一种模式，每次创建子 agent 都新建 session，不共享上下文，无 Fork、无文件系统邮箱、无 swarm 后端切换。

**四、工具编排**

Claude Code 的 toolOrchestration.ts 实现自动并行分区：每个工具声明 isConcurrencySafe()，框架自动合并安全工具为并行批次（最多 10 并发），写入操作走串行。BashTool 有 2600 行安全验证器（白名单检查 → AST 解析 → 20+ 专项验证器）。

![工具编排：自动并行 vs 手动批处理](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/a3c008fd954a458ea29f90ec3a5ac5a3~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667694&x-signature=Ri%2F4zigketBHnl6JrzM6RlLmL%2BI%3D)

OpenCode 用 Batch Tool 模式，模型需显式包数组调用（最多 25 并发），框架不做安全检查。bash 安全机制相对基础。

**五、效果差异根源（四因素叠加）**

- 上下文保真度：Claude Code 三层压缩确保长对话中模型看到高质量信息
- Prompt 工程深度：GrowthBook 特性开关做 A/B 测试迭代，精细到每个条件分支
- 工具执行效率：自动并行化反馈循环更快
- 缓存复用：Prompt cache + Fork 共享缓存叠加，延迟和成本更低

![同模型效果差异根源分析](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/3fa3dc2b860f4e44871488871d25e8c3~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667694&x-signature=NsJH6VCe5uLUy76Ps%2Br5FXnsopk%3D)

