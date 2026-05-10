---
ingested: 2026-05-09
wiki_pages: [ai-agent/agent-harness-overview]
---

# Harness Engineering总结

**文章信息**
- **原文标题**：Anthropic说：不要在等下一代模型了，立刻马上做Harness！
- **来源**：<https://www.toutiao.com/article/7621546818609955370>
- **作者**：编程进阶社
- **发表时间**：2026年3月26日 20:56

**一段话总结**：AI 编程领域正从"写好提示词"进化到"搭建运行环境"。2026 年 2 月，Mitchell Hashimoto 提出 Harness Engineering 概念——核心理念是：每当你发现 Agent 犯错，就花时间工程化一个解决方案，让它再也不会犯同样的错。Anthropic、OpenAI、Stripe、Cursor 等公司几乎同时走向了同一方向：OpenAI Codex 团队 5 个月生成 100 万行代码、人类零编码；Stripe 每周无人值守合并 1300+ PR；Anthropic 发现同一模型只换 Harness 成功率从 42% 跳到 78%。但 Harness 不是一劳永逸的——每一代新模型出来，旧约束可以拆掉，但新的更高阶约束空间会打开。反对者（如 OpenAI 的 Noam Brown）认为 Harness 只是拐杖，模型终将超越它。Anthropic 的回应：Harness 的可能性空间不会随着模型进步而缩小，它只是在平移。真正稀缺的能力，不在模型里面，在模型外面，而且每隔几个月就得重写一次。

---

**核心事件**：Anthropic 发布工程博客分享 Harness Engineering 实践，标志着这项技术进入成熟阶段。核心结论：同一个模型，只换外面的运行环境（Harness），编程基准成功率从 42% 跳到 78%——换壳带来的提升相当于换一代模型。

![Solo vs Harness 对比](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/dc5c4028af504b1dac56a2962de9c772~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775391279&x-signature=26sFzbSmnnRtXObGzymcNrf6Nv0%3D)

现在业内共识，当一个技术被 Anthropic 以博客的形式分享出来，说明这个技术是必需，且成熟的方案了。

**三代进化：Prompt → Context → Harness**

- **2022-2024 · Prompt Engineering**：核心关注怎么写好一条指令（few-shot、chain-of-thought、角色扮演），本质是打磨单次输入。
- **2025 · Context Engineering**（Andrej Karpathy & Shopify CEO Tobi Lütke 推出）：单条 prompt 不够了，需为模型动态构建整个上下文——相关文件、历史对话、工具定义、知识库检索结果。
- **2026年2月 · Harness Engineering**（Mitchell Hashimoto 提出）：搭建整个工作环境——让 Agent 能持续、稳定、高质量地工作。包含上下文工程，但在更高层面运作：约束、反馈循环、架构规则、工具链、生命周期管理。

![Harness Engineering 概念图](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/2e65fae269344f66b990ab4e5c31f010~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775391279&x-signature=P%2BPeYRqwXwJab6tYtgZcwD%2BbIDk%3D)

打个比方：Prompt Engineering 是写好一封邮件，Context Engineering 是把相关附件都带上，Harness Engineering 是搭建整个工作环境。

![三代进化：Prompt → Context → Harness](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/cfacb9c9709f43ada9a4eafca02aa481~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775391279&x-signature=11zoaMPEGhi29NbNYlL2qg80UhI%3D)

这个词最早来自 Mitchell Hashimoto——HashiCorp 联合创始人、Terraform 的缔造者。他今年 2 月写了篇博客，把自己使用 AI 编程的进化拆成了六个阶段。第五个阶段叫 Engineer the Harness。

**Harness Engineering 的定义**：

> 每当你发现 Agent 犯了一个错误，你就花时间去工程化一个解决方案，让它再也不会犯同样的错。

他在 Ghostty 项目里实践了这个理念——AGENTS.md 文件里的每一行规则，背后都对应一个 Agent 曾经犯过的错。他说，那个文件里的每一行都基于一次 Agent 的不良行为，而且几乎完全解决了这些问题。

**百万行代码的验证**

OpenAI 的 Codex 团队做了一个实验，让整个行业重新理解"工程师"这个角色。从一个空的 git 仓库开始，5 个月，大约 100 万行代码，1500 个 PR，全部由 Agent 生成。人类一行代码都没写。

团队一开始只有 3 个工程师，后来扩到 7 个。用 GPT-5 驱动的 Codex CLI 从零构建了一个完整的生产级应用。平均每位工程师每天合并 3.5 个 PR。如果用传统方式手写，工期大概是现在的 10 倍。

核心工程师 Ryan Lopopolo 写了一句话：**Agent 不难，Harness 才难。**

5 个月实战里他们总结了几条硬规则：
1. 仓库是 Agent 唯一的知识来源
2. 代码不仅要对人类可读，更要对 Agent 可读
3. 架构约束不靠 prompt，靠 linter
4. 自主性得一步步给
5. 如果 PR 需要大改才能合并，问题不在 Agent，在 Harness

OpenAI 团队给自己的角色总结是：**我们现在最大的挑战，在于设计环境、反馈循环和控制系统。不写代码了。写规则。**

Cursor 团队也做过一次长程的 Coding 实验，他们发现一个反直觉的问题：**约束解空间，反而让 Agent 更有生产力。** 当模型可以生成任何东西时，会浪费 token 探索死胡同。当 Harness 定义了清晰的边界，Agent 反而更快收敛到正确答案。

**不止 OpenAI——多家公司同时走向同一方向**

**Stripe** 的内部系统「Minions」，每周合并 1300 多个 PR，全部由无人值守的 Agent 完成。它的架构有个值得注意的设计：Blueprint 编排系统，把工作流拆成确定性节点和 Agentic 节点。

![Stripe Blueprint 编排架构](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/0343e6528e9c4124a0a594e634087f44~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775391279&x-signature=O4ZEFPdyrsP4SV%2B7JDAXR2Pxa6s%3D)

确定性节点——跑 linter、推送更改——按固定路径执行，不调用大模型。Agentic 节点——实现功能、修复 CI——让模型判断。

Stripe 还有一个硬规则：CI 最多跑两轮。第一轮失败，Agent 自动修复再跑一次。还失败，直接转交人类。不允许 Agent 在 CI 上无限重试。

他们内部的工具平台挂了大约 500 个 MCP 工具，但给每个 Agent 的只是精心筛选过的子集。因为他们发现了一条规律：**更多的工具并不等于更好的表现。**

Stripe 工程团队的总结：**成功取决于可靠的开发者环境、测试基础设施和反馈循环，跟模型选择关系不大。**

**Cursor** 的「Self-Driving Codebases」研究走得更远——每小时约 1000 个 commit，一周超过 1000 万次工具调用，启动后无需人工干预。但他们走过的弯路，本身就是 Harness 设计难度的证明：

![Cursor 五代架构迭代](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/6c1022bb3a5b4f119b47a7c16b9eedc2~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775391279&x-signature=z7vv3k%2BVjid6VaJ%2Bo0NUYPZT3oE%3D)

第一版单 Agent，复杂任务扛不住。第二版多 Agent 共享状态文件，锁竞争严重，Agent 之间互相打架。第三版结构化角色分工，太僵硬。第四版持续执行器，角色过载。最终版本是递归 Planner-Worker 模型。

他们还发现了一个有点黑色幽默的现象：一条模糊的初始指令，在数百个并发 Agent 之间会被放大。一个 Agent 犯的错，乘以几百个并发。结果可想而知。

**模型不会反省——Harness 存在的底层原因**

Anthropic 的关键发现：**模型不会评价自己的工作。** 让 Agent 自己评估刚写完的东西，它会自信地表示写得很好——即使在人类看来，质量明显不行。这个问题在主观任务上（比如前端设计，页面到底好不好看）尤其严重，因为没有二元的对错标准。但即使在有客观标准的任务上，Agent 的自我判断也会出问题。

这才是 Harness 需要存在的底层原因之一。模型能力够了，但它对自己的能力没有准确的认知。

Anthropic 的解法借鉴了 GAN（生成对抗网络）的思路：把生成和评估拆成两个独立的 Agent。一个 generator 负责写，一个 evaluator 负责评。evaluator 不是看截图打分，而是用 Playwright 真的去点页面、查 API、看数据库状态，像一个真人 QA 一样操作完再给反馈。

但 evaluator 也不是天生就靠谱。**开箱即用的 Claude 是一个很差的 QA Agent。** 早期的 evaluator 会发现问题，然后说服自己这不是大问题，接着就批准了。它也倾向于做表面测试，不去探边界。他们花了好几轮来校准 evaluator 的判断标准，让它的严苛程度符合人类预期。

关键在于，**让一个独立的 evaluator 变得严格，远比让 generator 学会自我批评容易得多。** 这就是拆分的价值。

他们做了一些全栈开发测试，用一套 3 个 Agent 的架构：planner 把一句话需求展开成完整的产品规格，generator 按 sprint 逐个功能实现，evaluator 在每个 sprint 结束后真机验收。

Solo 模式的输出看起来不错，但是功能都是坏的，界面完全看不出来。

![Solo 模式输出：游戏画面（核心功能不响应）](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/5a909368d1bd44468f5485cc88d4b2db~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775391279&x-signature=MHxQbJX7OZ438nUffX6pWrEi8iA%3D)

Harness 模式出来的版本，编辑器更完整，游戏能玩，物理引擎有些粗糙但核心交互是通的。

![Harness 模式输出：游戏可以玩](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/a3bbbfddcf59482ca1158e2fa56fbf34~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775391279&x-signature=odgUqf1n%2F1HHm1LPoKyiBsBOya4%3D)

evaluator 的 bug 报告精确到哪行代码出了什么问题。

**数据调研：优化壳的回报率可能比等下一代模型更高**

- **LangChain**：同一模型（gpt-5.2-codex），只改 Harness，Terminal Bench 2.0 成绩从 52.8% 升到 66.5%，排名从三十名开外直接进前五
- **Nate B Jones**：同一模型，Harness 不同，成功率 42% vs 78%
- **Terminal Bench 2.0**：Claude Opus 4.6 在 Claude Code Harness 上排名第 33，换一个 Harness 冲到第 5，同一个模型排名差了 28 位
- **Pi Research**：一个下午内，仅通过修改 Harness，提升了 15 个不同 LLM 的编程能力

**反对声音：Harness 是拐杖，终将被淘汰**

OpenAI 的 Noam Brown 直言：**Harness 就像一根拐杖，我们终将超越它。** 他的论据：推理模型出现前，开发者在 GPT-4o 上搭了大量复杂 Agentic 系统来模拟推理能力（路由器、编排器、multi-agent 协作），推理模型一出，这些东西一夜之间不需要了，反而让效果更差。

他给开发者的建议：**别花六个月搭建一个可能六个月后就被淘汰的东西。**

METR 的数据也给了 Harness 阵营一记闷击：他们找了 scikit-learn、Sphinx、pytest 三个项目的 4 位活跃维护者，审查了 296 个 AI 生成的 PR。维护者的合并率大约只有自动化评分通过率的一半。自动评分器说 Agent 能独立完成约 50 分钟的任务，但维护者实际愿意合并的 PR 对应的任务范围只有 8 分钟左右——7 倍的能力高估。

![METR 数据：自动评分 vs 维护者实际合并率（7 倍能力高估）](https://p3-sign.toutiaoimg.com/tos-cn-i-ezhpy3drpa/75a19aadb9b946b38600f5c03dc65c5b~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775391279&x-signature=D3N7i7VYuj8BL%2FuA5w1C9IM57yQ%3D)

Latent Space 给这一派起了个名字，叫 Bitter Lesson 阵营。出自 Rich Sutton 那篇著名的 AI 随笔：别在工程花活上下太多功夫，算力的增长终究会碾平一切。

**但壳在进化——Harness 的可能性空间不会缩小，只会平移**

Noam Brown 说得对不对？对了一半。Anthropic 自己的实验就是最好的证据。

他们最早的 harness 用 Opus 4.5，需要把工作拆成 sprint，一个 sprint 做一个功能，每个 sprint 之间做 context reset。因为 4.5 有一种"上下文焦虑"，当它感觉自己快用完上下文窗口时，会提前开始收尾工作，质量急剧下降。整套 sprint + context reset 的机制就是为了对付这个问题。

然后 Opus 4.6 出来了。这个版本规划更谨慎、长任务更稳定、长上下文检索更好、自己 debug 的能力更强。于是 Anthropic 直接砍掉了 sprint 结构，让 generator 连续跑完整个构建——一次跑了两个多小时，没有崩溃。

Sprint 被淘汰了。Noam Brown 说的"拐杖"，在这个案例里确实被扔掉了。

但 evaluator 没有被扔掉。因为即使 Opus 4.6 变强了，它的能力边界只是往外推了一些，边界本身并没有消失。在那个边界以内的任务，generator 可以独立搞定，evaluator 变成了多余的开销。但在边界附近的任务上，evaluator 依然能抓出 generator 自己看不到的问题——功能只实现了表面、API 路由定义顺序导致的隐蔽 bug、交互逻辑的缺失。

Anthropic 给出的判断是：**harness 的可能性空间不会随着模型进步而缩小。它会平移。**

模型变强了，旧的约束可以拆掉，但新的、更高阶的约束空间打开了。以前你要教 Agent 怎么管理上下文窗口，现在不用了；但你可以开始让它做四小时的自主开发任务，那就需要新的反馈和验收机制。

不只是 Anthropic 在做这件事。Manus 6 个月内重构了 5 次 Harness。LangChain 一年内重新架构了 3 次研究型 Agent。Vercel 砍掉了 80% 的 Agent 工具。

这说明一件事：**Harness 不是一次性工程，它是一个持续演化的系统。**

Anthropic 那篇博客的最后一句话是这么写的：

> 有意思的 harness 组合空间不会随着模型进步而缩小。它只是在移动。而 AI 工程师真正要做的，是持续找到下一个有效的组合。

**写在最后**

OpenAI 的 Codex 团队不写代码了，写的是架构规则、linter 配置和 AGENTS.md。Stripe 的工程师不写代码了，写的是 Blueprint 编排和 CI 限速策略。Anthropic 的工程师不写代码了，写的是 evaluator 的打分标准和校准逻辑。

**写代码正在变成一件成本很低的事。设计那套让 Agent 持续、稳定、高质量写代码的系统，才是真正贵的部分。** 而这套系统本身也不是一劳永逸的。每一代模型出来，都得重新审视哪些约束还管用，哪些该拆掉，哪些新的空间被打开了。

**真正稀缺的能力，不在模型里面，在模型外面。而且它每隔几个月就得重写一次。**


