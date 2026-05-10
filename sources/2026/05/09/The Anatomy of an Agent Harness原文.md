---
ingested: 2026-05-09
wiki_pages: [ai-agent/agent-harness-overview]
---

**文章信息**
- **原文标题**：The Anatomy of an Agent Harness
- **来源**：<https://blog.langchain.com/the-anatomy-of-an-agent-harness/>
- **作者**：Vivek Trivedy
- **发表时间**：未明确标注（LangChain 官方博客）

# Agent Harness 的剖析

**摘要**：Agent = Model + Harness。Harness 工程（Harness Engineering）就是我们围绕模型构建系统、将模型变成工作引擎的方式。模型包含智能，Harness 让智能变得有用。本文定义了 Harness 是什么，并推导出当今和未来 Agent 所需的核心组件。

## 请给 Harness 一个定义

**Agent = Model + Harness**

**如果你不是模型，那你就是 Harness。**

Harness 是模型之外的一切代码、配置和执行逻辑。原始模型不是 Agent，但当 Harness 给它提供状态（State）、工具执行（Tool Execution）、反馈循环（Feedback Loops）和可强制执行的约束（Enforceable Constraints）时，它就变成了 Agent。

具体来说，Harness 包括：

- **系统提示词（System Prompts）**
- **工具、技能、MCP 及其描述（Tools, Skills, MCPs + and their descriptions）**
- **内置基础设施（Bundled Infrastructure）**：文件系统、沙箱、浏览器
- **编排逻辑（Orchestration Logic）**：子 Agent 生成、交接、模型路由
- **钩子/中间件（Hooks/Middleware）**：用于确定性执行——压缩、续写、Lint 检查

在模型和 Harness 之间划分 Agent 系统边界的方式有很多，但在我看这是最清晰的定义，因为它迫使我们围绕模型智能来思考系统设计。

本文将逐一拆解 Harness 的核心组件，从模型这个核心原语出发，反推每个组件存在的原因。

## 为什么需要 Harness？——从模型的视角

有些事我们希望 Agent 能做，但模型本身做不到。Harness 就是为此而生。

模型主要接收文本、图像、音频、视频等数据输入，然后输出文本。仅此而已。开箱即用时，它们不能：

- 在交互中保持持久状态（Durable State）
- 执行代码
- 获取实时知识
- 搭建环境并安装依赖来完成工作

这些都是 Harness 层面的能力。LLM 的结构决定了需要某种机制将它们包裹起来才能做有用的工作。

例如，要实现"聊天"这样的产品体验，我们用一个 while 循环包裹模型来追踪历史消息、追加新用户消息。每个读到这里的人都已经用过这种 Harness。核心思想是：**我们希望把期望的 Agent 行为转化为 Harness 中的实际功能。**

## 从期望的 Agent 行为反推 Harness 工程

Harness 工程帮助人类注入有用的先验知识（Priors）来引导 Agent 行为。随着模型越来越强大，Harness 被用来精准地扩展和纠正模型，完成以前不可能的任务。

我们不会列举 Harness 的全部功能清单。目标是从"帮助模型做有用工作"这个出发点推导出一组核心功能。我们遵循的模式是：

**我们想要的行为（或想要修复的问题）→ 帮助模型实现此目标的 Harness 设计。**

## 文件系统：持久化存储与上下文管理

**我们希望 Agent 拥有持久化存储，能对接真实数据、卸载放不进上下文的信息，并跨会话保留工作成果。**

模型只能直接操作上下文窗口中的内容。在有文件系统之前，用户必须把内容复制粘贴给模型——这是笨拙的 UX，对自主 Agent 来说更行不通。世界本来就在用文件系统做工作，模型自然也在数十亿 token 的训练数据中学会了如何使用它们。自然而然的解决方案是：

**Harness 配备文件系统抽象和文件操作工具。**

文件系统可以说是最基础的 Harness 原语，因为它解锁了大量能力：

- Agent 拥有工作空间，可以读取数据、代码和文档
- 工作可以增量添加和卸载，而不是把所有东西都塞进上下文。Agent 可以存储中间输出，并维护跨会话的持久状态
- 文件系统是天然的协作接口。多个 Agent 和人可以通过共享文件协调工作。Agent Teams 等架构依赖于此

Git 为文件系统增加了版本控制，使 Agent 能追踪工作进度、回滚错误、分支实验。后文还会继续讨论文件系统，因为它是一个关键的 Harness 原语，支撑着其他功能。

## Bash + 代码执行：通用问题解决工具

**我们希望 Agent 能自主解决问题，而不需要人预先设计每一个工具。**

当前主要的 Agent 执行模式是 [ReAct 循环](https://docs.langchain.com/oss/python/langchain/agents?ref=blog.langchain.com)，模型推理、通过工具调用执行动作、观察结果，然后在 while 循环中重复。但 Harness 只能执行它有逻辑的工具。与其强迫用户为每个可能的动作构建工具，更好的方案是给 Agent 一个通用工具，比如 Bash。

**Harness 配备 Bash 工具，使模型能通过编写和执行代码自主解决问题。**

Bash + 代码执行是给模型一台"计算机"、让它们自主解决剩余问题的一大步。模型可以通过代码即时设计自己的工具，而不限于预配置的固定工具集。

Harness 仍然提供其他工具，但代码执行已经成为自主问题解决的默认通用策略。

## 沙箱与工具：执行和验证工作

**Agent 需要一个具备合理默认配置的环境，以便安全地行动、观察结果并取得进展。**

我们给了模型存储和代码执行能力，但所有这些都需要在某个地方运行。在本地执行 Agent 生成的代码有风险，而且单一本地环境无法扩展到大规模 Agent 工作负载。

**沙箱为 Agent 提供安全的运行环境。** Harness 可以连接到沙箱来运行代码、检查文件、安装依赖和完成任务，而不是在本地执行。这创造了安全隔离的代码执行。为增强安全性，Harness 可以对命令进行白名单控制并强制网络隔离。沙箱还能解锁扩展性——环境可以按需创建、跨多个任务并行展开、工作完成后销毁。

好的环境需要好的默认工具。Harness 负责配置工具，让 Agent 能做有用的工作，包括预装语言运行时和包、Git 和测试的 CLI、用于网页交互和验证的[浏览器](https://github.com/vercel-labs/agent-browser?ref=blog.langchain.com)。

浏览器、日志、截图和测试运行器等工具让 Agent 能观察和分析自己的工作，帮助它们创建**自我验证循环**（Self-Verification Loops）：编写应用代码→运行测试→检查日志→修复错误。

模型开箱即用不会配置自己的执行环境。决定 Agent 在哪里运行、有哪些工具可用、能访问什么、如何验证工作——这些都是 Harness 层面的设计决策。

## 记忆与搜索：持续学习能力

**Agent 应该记住它见过的东西，并访问训练时不存在的信息。**

模型除了权重和当前上下文中的内容外，没有额外知识。在无法编辑模型权重的情况下，"添加知识"的唯一方式是**通过上下文注入**。

对于记忆，文件系统再次成为核心原语。Harness 支持 [AGENTS.md](http://agents.md/?ref=blog.langchain.com) 等记忆文件标准，这些文件在 Agent 启动时被注入上下文。当 Agent 添加和编辑这个文件时，Harness 将更新后的文件加载进上下文。这是一种[持续学习](https://www.ibm.com/think/topics/continual-learning?ref=blog.langchain.com)的形式——Agent 将一个会话中的知识持久存储，并注入到未来的会话中。

知识截止日期意味着模型无法直接获取新数据（如更新的库版本），除非用户直接提供。对于最新知识，Web Search 和 [Context7](https://context7.com/?ref=blog.langchain.com) 等 MCP 工具帮助 Agent 获取知识截止日期之后的信息，如新的库版本或训练时不存在的当前数据。

**Web Search 和查询最新上下文的工具是值得内建到 Harness 中的有用原语。**

## 对抗上下文衰减

**Agent 的表现不应该在工作过程中变差。**

[Context Rot（上下文衰减）](https://research.trychroma.com/context-rot?ref=blog.langchain.com)描述了模型如何随着上下文窗口填满而在推理和任务完成方面表现变差。上下文是一种珍贵且稀缺的资源，因此 Harness 需要策略来管理它。

**当今的 Harness 本质上是良好上下文工程的交付机制。**

**压缩（Compaction）** 解决上下文窗口接近填满时的问题。不压缩的话，当对话超过上下文窗口会发生什么？一种可能是 API 报错——这不好。Harness 必须对此使用某种策略。压缩智能地卸载并总结现有上下文窗口，使 Agent 能继续工作。

**工具调用卸载（Tool Call Offloading）** 有助于减少大型工具输出的影响，这些输出会嘈杂地充斥上下文窗口而提供无用信息。Harness 对超过阈值的工具输出保留头部和尾部 token，将完整输出卸载到文件系统，以便模型在需要时访问。

**技能（Skills）** 解决了启动时加载过多工具或 MCP 服务器导致性能下降的问题。技能是一种 Harness 原语，通过**渐进式披露**（Progressive Disclosure）来解决此问题。模型并没有选择在启动时将 Skill 前言加载到上下文中，但 Harness 可以支持这一点来保护模型免受上下文衰减的影响。

## 长期自主执行

**我们希望 Agent 能自主地、正确地、在长时间跨度内完成复杂工作。**

自主软件创建是编码 Agent 的圣杯。但当今的模型存在提前停止、分解复杂问题困难、以及工作跨越多个上下文窗口后的不连贯问题。一个好的 Harness 必须围绕所有这些来设计。

这时前面的 Harness 原语开始产生复合效应。长期工作需要持久状态、规划、观察和验证，以在多个上下文窗口中保持工作连贯。

**文件系统和 Git 用于跨会话追踪工作。** Agent 在长任务中产生数百万 token，文件系统持久捕获工作以跟踪进度。加入 Git 后，新 Agent 可以快速了解项目的最新工作和历史。对于多个 Agent 协同工作，文件系统还充当**共享工作台账（Shared Ledger）**。

**Ralph 循环用于延续工作。** [Ralph 循环](https://ghuntley.com/loop/?ref=blog.langchain.com)是一种 Harness 模式，通过钩子拦截模型的退出尝试，在干净的上下文窗口中重新注入原始提示词，迫使 Agent 针对完成目标继续工作。文件系统使这成为可能——每次迭代都以新鲜上下文开始，但从上一次迭代读取状态。

**规划和自我验证以保持正轨。** 规划是模型将目标分解为一系列步骤。Harness 通过良好的提示词和在文件系统中注入如何使用计划文件的提醒来支持此功能。完成每一步后，Agent 受益于通过自我验证检查工作正确性。Harness 中的钩子可以运行预定义的测试套件，并在失败时将错误消息循环回模型；或者提示模型独立地自我评估代码。**验证将解决方案锚定在测试中，并为自我改进创建反馈信号。**

## Harness 的未来

### 模型训练与 Harness 设计的耦合

当今的 Agent 产品（如 Claude Code 和 Codex）在训练时将模型和 Harness 同时纳入循环。这帮助模型提升在 Harness 设计者认为模型应天生擅长的动作上的能力，如文件系统操作、Bash 执行、规划、或用子 Agent 并行化工作。

这形成了一个反馈循环：发现有用的原语→添加到 Harness→用于训练下一代模型。随着这个循环重复，模型在它们被训练的 Harness 中变得越来越强大。

但这种共同进化对泛化性有有趣的副作用。改变工具逻辑会导致模型性能下降。一个好例子是 [Codex-5.3 提示指南](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide/?ref=blog.langchain.com#apply_patch)中描述的 `apply_patch` 工具逻辑。真正智能的模型应该能轻松切换补丁方法，但在 Harness 中训练就造成了这种过拟合。

但这并不意味着你任务的最佳 Harness 就是模型在训练时使用的那个。[Terminal Bench 2.0 排行榜](https://www.tbench.ai/leaderboard/terminal-bench/2.0?ref=blog.langchain.com)就是一个好例子。Claude Code 中的 Opus 4.6 得分远低于其他 Harness 中的 Opus 4.6。在[之前的博客](https://x.com/Vtrivedy10/status/2023805578561060992?s=20&ref=blog.langchain.com)中，我们展示了仅通过改变 Harness，就将我们的编码 Agent 从 Terminal Bench 2.0 的第 30 名提升到第 5 名。**为你的任务优化 Harness 还有很大空间可挖。**

### Harness 工程的走向

随着模型变得更强大，今天 Harness 中的部分功能将被模型自身吸收。模型在规划、自我验证和长期连贯性方面将更加原生可靠，因此对上下文注入等的需求会减少。

这似乎意味着 Harness 会变得不那么重要。但就像提示工程今天仍然有价值一样，Harness 工程很可能仍对构建好的 Agent 有用。

诚然，今天的 Harness 在弥补模型缺陷，但它们也在围绕模型智能构建系统以使其更有效。一个配置良好的环境、合适的工具、持久的状态和验证循环，使任何模型都更高效——无论其基础智能如何。

Harness 工程是一个非常活跃的研究领域，我们在 LangChain 用它来改进 Harness 构建库 [deepagents](https://docs.langchain.com/oss/python/deepagents/overview?ref=blog.langchain.com)。以下是我们当前探索的几个开放且有趣的课题：

- 编排数百个 Agent 在共享代码库中并行工作
- Agent 分析自己的轨迹（Traces）来识别和修复 Harness 层面的失败模式
- Harness 根据任务即时动态组装合适的工具和上下文，而非预配置

本文是一次定义 Harness 是什么以及它如何被我们期望模型完成的工作所塑造的尝试。

**模型包含智能，Harness 是让智能变得有用的系统。**

**致敬更多 Harness 构建、更好的系统、更好的 Agent。**

