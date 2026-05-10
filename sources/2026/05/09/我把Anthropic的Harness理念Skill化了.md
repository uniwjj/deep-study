---
ingested: 2026-05-09
wiki_pages: [ai-agent/harness-build-to-delete]
---
## Harness Skill化

**文章信息**
- **原文标题**：我把Anthropic的Harness理念Skill化了
- **来源**：<https://www.toutiao.com/article/7621738959667757609>
- **作者**：VibeCoder
- **发表时间**：2026-03-27 09:23

**一段话总结**：作者基于 Anthropic 2026年3月24日发布的 Harness 工程博客，在自己的 harness 工具库中实现了 `/baseline`、`/evaluate`、`/eval-fix` 三条指令，核心理念是将 Generator（写代码的 Agent）和 Evaluator（验收的 Agent）拆成两个独立角色进行对抗评估，解决 Agent 自我评价偏差（Self-Evaluation Bias）的问题。在复刻桌面 AI Agent 助手 QoderWork 的实战中，通过三轮对抗评估修复，将复刻度从 5.9/10 提升至 9.8/10，修复了 30 多个 UI 差异和 3 个运行时 bug（JSON 半括号泄露、空白 markdown 区块、聊天区被底部栏遮挡）。作者的核心判断：写代码的 Agent 已经很多了，但能验收代码的 Agent 仍然稀缺，Harness 工程可能成为 Agent 领域的标准实践。

---

**核心事件**：Anthropic 提出 Harness 理念——Agent 不难，驾驭 Agent 的编排框架（Harness）才难，并通过 Generator vs Evaluator 对抗机制解决 AI 自我评估不可靠的问题。

Anthropic 3月24日发了一篇工程博客 "Harness design for long-running application development"，核心论点用一句话概括：**Agents aren't hard; the Harness is hard.**

Agent 本身能力已经很强，难的是怎么驾驭它——什么时候扩展上下文、什么时候压缩、什么时候让多个 Agent 分工、什么时候该停下来。这些编排逻辑加在一起，就是 Harness。

他们跑了个实验：让 Claude 独立写一个全栈 Web 应用，单跑一次 20 分钟花 9 美元，结果核心功能都是坏的。加上完整的 Harness（Planner + Generator + Evaluator 三 Agent 体系），跑了 6 小时花 200 美元，但产出了一个完整可用的应用。

这里面 Evaluator 的作用很关键。它用 Playwright MCP 控制浏览器，实际点页面、截图、打分，发现 Generator 自己完全看不到的问题。Anthropic 的原话是：**Tuning evaluator skepticism is more tractable than making generators self-critical.** 与其教 Generator 学会自我批评，不如把 Evaluator 的怀疑精神调到位。

![Harness 评估脚本 evaluate.md](https://p26-sign.toutiaoimg.com/tos-cn-i-axegupay5k/859849f017774d33a94b049fc43d8c87~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775393166&x-signature=B2Bf36ODXUQnHPr2Urk1sn2Aquc%3D)

作者基于这个理念，在 harness 代码库里实现了三个指令：

**/baseline** — 基线采集。Evaluator 用 Playwright MCP 系统化探索参照产品（QoderWork），逐页截图，逐功能记录交互步骤，最终输出一份完整的 feature inventory。

**/evaluate** — 对抗评估。拿着基线逐功能对比两个产品，评分体系是 4 个维度加权：功能完整性 40%、交互一致性 25%、视觉保真度 20%、技术质量 15%。评分校准很严格：10 分 = 与参照完全不可区分，7 分 = 功能正常但有明显差异，4 分以下 = 基本不能用。每个差异都要截图取证，不接受"按钮看起来不太一样"这种模糊描述。

**/eval-fix** — 修复循环。GAN 风格的对抗 loop：Generator 根据 eval-report 修 bug → Evaluator 重新跑评估 → 分数涨了继续、连续两轮不涨报告 stagnation、分数跌了就 revert。内置三种终止条件：收敛（≥7 分）、停滞（连续 2 轮没进步）、回退（分数下降），不会无限循环。

![QoderWork 界面 — 被复刻的参照产品](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/fc367b5ce37742378939e063943c385e~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775393166&x-signature=OFQsTJn7o63YYFe7rc5is3H7pgw%3D)

QoderWork 是一个桌面 AI Agent 助手，类似 Claude Code 的 GUI 版本。作者用 WeWork 项目复刻它，基线采集用了 8 张全流程截图 + 一份 135 行的 elements.json。

![QoderWork 任务执行功能](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/3dca8ee2a38f47389d97d9692c8bf314~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1775393166&x-signature=gTQtYgAMR44EdhhSmuT%2BeUkkSN8%3D)

**三轮对抗评估迭代过程：**

第一次 /evaluate 评分 **5.9/10**，问题列表：用户消息气泡样式完全错误（灰色左对齐 vs 应该是深色右对齐）、定时任务页是列表布局而不是 2x2 卡片网格、状态栏颜色用了红色（QoderWork 用绿色）、侧边栏缺少折叠按钮、顶栏结构不对。修了一轮分数到 **8.2/10**。

第二次 evaluate 暴露了更深层的运行时 bug——这些在静态截图对比中看不出来：

- **Bug 1：{\" 半括号泄露**。LLM 流式输出 JSON 控制指令时，前几个字符 {\" 先到了，但 cleanLLMText 的正则只匹配完整的 JSON，直接渲染到了聊天界面。修复：加通用尾部正则 `/\{\"[^}]*$/g`。
- **Bug 2：空白 markdown 区块**。每轮 LLM 调用开始时预创建了空的 text block，如果这轮只有工具调用没有文本输出，空块就留在界面上。三层防御：executor 层 lazy 创建 + 组件层空内容 return null + isSubstantial 门槛检查（>2 字符且含 \w）。
- **Bug 3：聊天区内容被底部栏遮挡**。固定定位的状态栏 + 输入框高度约 160px，但滚动区域 padding-bottom 只有 128px。修复加了动态 padding（运行时 180px / 空闲时 140px）+ scrollRef 自动跟随 + 用户上滚暂停手势检测。

修完第二轮分数到 **9.4/10**。第三轮将代码区块深色主题统一改成浅色卡片风格后达到 **9.8/10**。

**实操体会：**

- **基线采集的颗粒度决定评估上限。** 8 张截图加 elements.json 已覆盖主流程，但边缘状态（空数据、加载中、错误态）缺失导致这些区域没法评估。
- **Evaluator 发现的问题和开发者自己看到的完全不一样。** 开发者改完代码看一眼"嗯差不多了"，但 Evaluator 截图对比后指出用户气泡对齐方式整个反了——因为开发者知道自己写了什么，但 Evaluator 只看结果。
- **流式渲染 bug 是截图对比抓不到的。** JSON 泄露、空白区块只在 Agent 运行过程中出现，说明 Evaluator 的 Playwright MCP 不能只截静态页面，还得模拟真实用户操作流。
- **对抗循环的 stagnation 检测很有用。** 3-4 轮就能收敛到 9+ 分，超过这个次数说明需要人工介入看看架构层面的差异。

**作者判断**：让 AI 评 AI 比让 AI 评自己靠谱得多。把生成和评估拆开，各自独立运行，用截图和评分做硬性对齐，这个方向可能会成为 Agent 工程中的标准实践。Harness 就是基于模型和业务抠流程细节，没有一招鲜的通用解法，需要根据业务需求细细打磨。开源项目地址：zxdxjtu/harness。

