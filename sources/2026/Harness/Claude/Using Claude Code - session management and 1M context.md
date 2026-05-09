---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-harness]
---

## Using Claude Code: session management and 1M context

**文章信息**：
- **原文标题**：Using Claude Code: session management and 1M context
- **来源**：https://claude.com/blog/using-claude-code-session-management-and-1m-context
- **作者**：Thariq Shihipar（Anthropic technical staff，Claude Code 团队）
- **发表时间**：2026 年 4 月 15 日

---

**一段话总结**：

本文是 Claude Code 会话管理的实践指南，核心议题由新命令 `/usage` 的发布引出。文章解释了上下文窗口、"上下文腐化"（context rot）和自动压缩（autocompact）的原理，并详细说明在每次对话结束后可选的五种操作：继续（continue）、回退（rewind）、清空（clear）、压缩（compact）、子代理（subagent）。关键结论是：新任务换新会话；走错路要用 rewind 而非追加纠错；长会话用 /compact 低成本精简；全新任务用 /clear 自己掌控上下文；大量中间输出只需最终结论时派发子代理。1M token 上下文不等于可以无限续命，上下文越长模型越分心，主动管理才是高效使用 Claude Code 的关键。

---

**作者判断**：

作者认为上下文管理方式对 Claude Code 使用体验影响极大，但大多数用户没有意识到这一点。核心立场是：**新任务 = 新会话**，这是最基本的经验法则。对于是否压缩还是清空，作者明确区分了两者的取舍——`/compact` 省力但由模型决定保留什么，`/clear` 费力但你决定带走什么。对于自动压缩失败的情况，作者坦承"模型在最需要压缩的时刻恰好处于最低性能点"，建议提前主动压缩，而不是等到被迫触发。

---

**核心内容**：

**上下文窗口与上下文腐化**

上下文窗口是模型在生成回复时能"看到"的全部内容：系统提示、对话历史、每次工具调用及其输出、所有已读文件。Claude Code 当前上下文窗口为 **100 万 token**。

"上下文腐化"（context rot）指随着上下文增长，模型性能会下降——注意力被分散到更多 token 上，旧的、不相关的内容开始干扰当前任务。

![上下文窗口示意图](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/69e02238a3e7e9532cb643de_image6.png)

**压缩（Compaction）**

当接近上下文窗口上限时，系统自动将已完成的任务压缩成摘要，模型在新上下文中继续工作，称为自动压缩（autocompact）。也可手动触发 `/compact`。

![压缩示意图](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/69e02297f13357d9b32d8312_image5.png)

**每次完成后的五种选择**

每次 Claude 完成任务后，可选的操作：

- **Continue**（继续）：发送下一条消息，适合上下文仍有用的情况
- **`/rewind`（双击 Esc）**：回退到某条消息，丢弃之后的内容，重新提示
- **`/clear`**：清空会话，开启全新会话
- **`/compact`**：将对话压缩成摘要后继续
- **Subagents**：将下一段工作委托给拥有独立上下文的子代理，只将结果带回

![五种选择示意图](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/69e022cf45e7f9c9d025756d_image3.png)

**何时开启新会话**

经验法则：**开始新任务时，同时开启新会话**。

1M 上下文让更长的任务（如从零构建全栈应用）变得更可靠，但上下文腐化依然存在。任务相关但上下文部分有用时（如为刚实现的功能写文档），可以不换会话，避免重读文件的开销。

**用 Rewind 而非追加纠错**

双击 Esc 或 `/rewind` 可跳回任意历史消息，其后的消息从上下文中删除。

典型场景：Claude 读了 5 个文件、尝试了某种方案、失败了。直觉是追加"那样不行，试 X"，但更好的做法是 rewind 到文件读取之后，重新提示："不要用方案 A，foo 模块不暴露那个接口——直接走方案 B"。

还可以用 `"summarize from here"` 让 Claude 总结当前发现，生成一份交接消息，相当于"来自未来的自己给过去的 Claude 的提示"。

![Rewind 示意图](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/69e0234c97977d4944bea810_image4.png)

**`/compact` 与 `/clear` 的区别**

| 操作 | 谁决定保留什么 | 特点 |
|------|--------------|------|
| `/compact` | 模型 | 省力；可以附加指令引导，如 `/compact focus on the auth refactor, drop the test debugging` |
| `/clear` | 你自己手写摘要 | 费力；你精确控制带走什么，上下文更干净 |

![compact 示意图](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/69e02427049669efd6bb7604_image1.png)

**自动压缩失败的原因**

当会话方向模型无法预判时，自动压缩质量最差。例如：漫长 debug 会话后触发 autocompact，摘要聚焦调试内容，而你下一条消息是"现在修 bar.ts 里那个警告"——那个警告可能已被压缩丢弃。

根本原因：**模型在最需要压缩的时刻（上下文最长），恰好是性能最低点**。有了 1M 上下文，建议提前主动 `/compact` 并附上接下来要做的描述，避免被迫触发。

**子代理（Subagents）与干净的上下文窗口**

子代理适合"知道某段工作会产生大量中间输出、但只需要最终结论"的场景。子代理拥有独立的新上下文窗口，完成工作后只将最终报告返回给父代理。

![子代理示意图](https://cdn.prod.website-files.com/68a44d4040f98a4adf2207b6/69e0241044643c5402b312a9_image2.png)

判断标准：*我还需要这个工具调用的输出，还是只需要结论？*

适合派发子代理的典型指令：
- "启动一个子代理，根据以下规格文件验证这项工作的结果"
- "派一个子代理读取那个代码库，总结它的 auth 流程实现，然后你用同样的方式实现"
- "派一个子代理根据我的 git 变更写这个功能的文档"

**情境选择速查表**

| 场景 | 推荐操作 | 原因 |
|------|---------|------|
| 同一任务，上下文仍然有用 | Continue | 窗口里的内容仍是必要的，不必重建 |
| Claude 走错了路 | Rewind（双击 Esc） | 保留有用的文件读取，丢弃失败尝试，用学到的内容重新提示 |
| 任务进行中但会话被调试/探索内容撑大 | `/compact <hint>` | 省力；Claude 决定什么重要，用指令引导方向 |
| 开始全新任务 | `/clear` | 零腐化；你精确控制带什么进去 |
| 下一步会产生大量输出但只需要结论（代码库搜索、验证、写文档） | Subagent | 中间工具噪音留在子代理上下文；只有结果返回 |
