---
title: 移动端 GUI 执行的 Agent Harness 探索
description: OPPO 研究院瞿祥谋：Harness 是模型与真实世界之间的受控运行时——五大能力（context 管理/权限管控/执行校验/状态观测/反思纠错），数据飞轮与异步真机训练架构（HammerEnv/Async Rollouter/GRPO），ColorGUI-32B 基准表现，以及稳定执行（异常感知/主动交互/个性化/环境标准化）的工程化路径
aliases: [移动端 GUI Agent, ColorGUI, ColorAgent, GUI Agent Harness, Mobile GUI Agent, OPPO GUI Agent]
tags: [ai-agent, practice, concept]
sources: [2026/08/11/Harness Engineering Agent执行与控制工程论坛/01-瞿祥谋-移动端 GUI 执行的 Agent Harness 探索.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 移动端 GUI 执行的 Agent Harness 探索

> 来源：2026 Agent 大会「Harness Engineering Agent执行与控制工程论坛」演讲《移动端 GUI 执行的 Agent Harness 探索》，讲者瞿祥谋（OPPO 研究院，邮箱 lokinko@oppo.com，团队 madeagent.ai）。

## 核心定义：Harness 是模型与真实世界之间的受控运行时

| | AGENT（发动机） | HARNESS（控制系统） |
|---|---|---|
| 决定 | 决定"想做什么" | 决定"能否做、如何做、如何确认" |
| 公式 | `action = agent(input)` | `action = harness(input, agent, env)` |

- Harness 是包裹在 Agent 外部、连接模型与真实环境的受控执行运行时
- 原则：**约束边界，不削弱智能；管控执行，不接管推理**
- 全栈闭环：环境 × 数据 × 训练 × 稳定执行，构成持续进化的全栈闭环

### 受控执行运行时五大能力

1. **context 管理**：用户意图分析，历史状态整合，上下文管理
2. **权限管控**：权限分级管控，敏感界面拦截
3. **执行校验**：模型动作校验，拦截非法动作
4. **状态观测**：记录可回放 trace、规则决策、环境状态变化
5. **反思纠错**：判断动作是否生效、任务是否进展、目标是否完成或已经失败

## 动机：一次 Demo，不等于可规模化能力

真正的工程目标：**在真实环境波动中，可复现地运行大量任务**。三组证据：

| 维度 | 数据 | 结论 |
|------|------|------|
| 模型上限 pass@k | 模型训练过程中 pass@k 波动（Pass@1 与 Pass@16，含 Reset Point） | 理论上限高、高波动性 |
| 稳定性 pass@k | z-bench 任务中，GPT-4o 的 pass@1 = 61.2%，pass@8 降至 25% | 稳定性和一致性难以满足，采样存在偶发性 |
| 长程任务 | 在 GUI-CEval 中，3 步成功率 58% → 5 步成功率 46% | 长程任务成功率持续衰减 |

> Agent Harness：在真实环境波动中，稳定可控地释放模型能力。

注：z-bench 引用原文首字母为花体数学字符，无法可靠辨认（此处按幻灯片视觉转写）；对应文献 Yao, Shunyu, et al., ICLR 2025。

## 为什么做移动端 GUI Agent

- 不用等待所有软件专门适配，AI 就可以先像人一样操作现有数字世界
- **为什么是移动端**：连续性、身份系统、物理世界
- **为什么是 GUI Agent**：看懂界面、理解目标、规划动作、确认进展
- 基本循环：Instruction / Context / Screenshot → **VLM**（OR 直接 Tool Calls：Browser Search / Code / API）→ **Action Sequence**（Click / Type / Swipe / Wait / Open）→ 新截图回环

三个结论句：
- 面向开放目标和动态路径，模型处理不确定性，不用等待所有软件适配，AI 可以直接操作现有数字世界
- 手机从"人操作一堆 App"，变成"人表达目标，系统里的 Agent 调度 App 和服务"
- 让人从数字世界的操作员，变成数字世界的委托人

## 移动端的不确定性

| 类别 | 内容 |
|------|------|
| **环境状态** | 登录、权限；版本更新、网络延迟；后台恢复、应用缓存 |
| **观测与动作** | 弹窗、动态广告；文本遮挡、键盘干扰；控件密集、动作时延 |
| **长链路任务** | 环境改变不可逆；长链路误差累积 |

配真实轨迹截图（蜜雪冰城外卖点单）：Step 2 (0.84s) 点击"外卖"按钮；Step 6 (0.89s) 输入品牌名 text: 蜜雪冰城；Step 7 (0.91s) 输入单品名 text: 满杯百香果；Step 9 (0.94s) 点击选规格按钮。

## 全链路技术栈总览

三大板块：**数据基建 | 模型训练 | Harness 框架**。两个交互目标：

- **Robust Agent-Environment Interaction**（鲁棒的 Agent-环境交互）：以"帮我点一杯瑞幸咖啡"为例——Retrieved knowledge（点餐要用美团 App）→ Thought（打开美团 App / 点击咖啡图标）→ Reflection（当前页面是 Cutti Coffee 而非 Luckin Coffee，需返回上一页）→ Thought（点击搜索按钮搜索 Luckin Coffee）→ Action: Click (851, 264)
- **Personalized Agent-User Interaction**（个性化的 Agent-用户交互）：以"来一杯美式"为例——Personalized Historical Trajectories（1. 星巴克冰美式：美团、大杯、不加糖；2. Cutti Coffee 冰拿铁：美团、大杯、热、30% 糖；3. 喜茶冰茶：美团、大杯、冰、50% 糖）→ User intent（用户偏好冰咖啡）→ 主动追问（要美式还是拿铁？）→ Action: Click (294, 1100) / Click (733, 2326) / 把温度从热改冰 Click (533, 1956)

## 数据飞轮：每次执行，都进入下一轮能力

模型训练优化：困难样本进入训练，新模型回到在线验证。

- 飞轮结构：**数据质量管理**（清洗、标注、归因、脱敏与版本化）→ trace → **offline agent / harness** → env → **在线数据采集**（任务执行产生状态、动作、反馈与结果）
- 数据飞轮形成对 agent 能力的及时反馈，通过失败归因持续迭代模型能力
- 三个标准：**任务覆盖**（App × 能力 × 难度）、**轨迹标准**（Milestone · Result）、**质量门禁**（合法性 · 一致性）。任务设计定义希望模型掌握的能力和覆盖范围；覆盖度决定边界，质量门禁决定模型真正学到什么
- 三条原则：数据质量决定训练质量，任务构建决定模型能力；离线训练困难样本优化模型性能，在线推理验证性能提升；在线训练适应环境变化和真实线上分布漂移，提升鲁棒性与泛化性

### 数据飞轮环境构建（异步训练架构）

两大挑战：**采样效率与训练速度严重失配**（手机 GUI 任务涉及频繁的动作执行与截图传输，传统同步训练导致 GPU 闲置率极高，算力浪费严重）；**真机环境极高的不可控性**（随机弹窗、系统通知、网络波动等干扰，对训练流程容错能力提出严苛挑战）。

异步架构（数据采样 + 奖励 → 消息队列 → 异步训练闭环）：

- **Reward System**：planner agent (VLM)、compressor agent (VLM)；outcome → reward → reward shaping → rewards → samples → Message Queue
- **Async Rollouter**（采样侧，参数同步标记 n*）：input context → VLLM（multi-turn processor）→ predicted action + screenshot → stepwise samples
- 真机环境链路：**HammerEnvClient (Gradio API)**（request device | step | get state）→ **HammerServer (Gradio Web Server)** → **ADB Bridge | Screenshot Capture | Action Execution** → **Android Devices (Cloud + Local)**，设备 D1…DN
- **Async Trainer**（训练侧，标记 k*）：trainer worker，算法 GRPO/GSPO，policy update；Message Queue → mini-batches 输入；**Parameter Synchronizer**（after m updates）把参数同步回 Rollouter

### 三步能力阶梯：离线提高筛选速度，在线证明真实价值

| STEP | 阶段 | 能力 | 说明 |
|------|------|------|------|
| STEP 1 | GROUNDING（看懂界面并对齐可交互对象） | SFT | 视觉编码捕获页面布局、图标、图片内容、选中状态和空间关系；Self-Adaptive Milestones Generation、Rubric Reward Curve、Reward Assignment（Success Traj. / Failure Traj. 分流） |
| STEP 2 | PLANNING（维护子目标与长程状态） | Offline-RL | 历史压缩 · 状态跟踪 · 重规划，持续维护任务状态，理解、规划问题解决路径 |
| STEP 3 | RECOVERY（无反馈 · 偏航 · 异常状态） | Online-RL | 恢复错误并确认目标满足；反思、纠错等能力的建立 |

奖励公式（幻灯片公式碎片）：R_total = λ1·R_format + A(t)·R_mil + R_outcome（原文为公式示意图，符号以幻灯片为准）。能力从「低阶」到「高阶」。

## 自动执行 Agent：ColorGUI-32B

基于 **Qwen3-VL-32B** 后训练：在保持基模通用能力的前提下，大幅强化对移动设备自动执行任务的理解，能解决高频、长尾的执行需求，具备长程任务理解与规划能力，稳定受控地在设备上执行任务。定位：中文手机 GUI Agent（理解指令 · 规划步骤 · 真机执行，ColorOS 真机）。

### 基准表现（Success Rate，%）

**AndroidWorld**：ColorAgent (Screenshot) **77.2**；MobileRL (Screenshot+XML) 75.8；UI-TARS-2 (Screenshot) 73.3；Mobile-Agent-v3 (Screenshot) 73.3；GUI-Owl-7B (Screenshot) 66.4；UI-Venus (Screenshot) 65.9；Qwen3-VL (Screenshot) 63.7；MobileUse (Screenshot) 62.9；Seed1.5-VL (Screenshot) 62.1；V-Droid (XML) 59.5；Agent S2 (Screenshot) 54.3；UI-TARS (Screenshot) 46.6

**AndroidLab**：ColorAgent (Screenshot) **50.7**；MobileRL (Screenshot+XML) 46.8；MobileUse (Screenshot) 44.2；UI-Genie-Agent (Screenshot+XML) 41.2；V-Droid (XML) 38.3；AutoGLM (Screenshot+XML) 36.2；GPT-4o (Screenshot) 31.2；GPT-4o (XML) 25.4；LLaMA3.1-8B-ft (XML) 23.9；GLM4-9B-ft (XML) 21.0；Gemini-1.5-Pro (XML) 18.8；Gemini-1.5-Pro (Screenshot) 16.7

任务示例（ColorBench 类长程任务）：热点浏览与总结（打开微博热搜，浏览当前榜单，并总结热点内容）、内容搜索与连续操作（在小红书搜索"减脂餐"，找到食谱笔记并收藏，再搜索健身博主并关注两位）、本地生活信息查询（在美团搜索米线，找到销量最高的第一家店铺并查看用户评价）、跨路线行程查询（在携程旅行同时查询明天深圳到北京、广州到佛山的火车票）、跨应用内容比对（分别在爱奇艺和腾讯视频搜索《甄嬛传》，对比哪个平台可以观看）。

## 稳定执行（Control → Detect → Recover）

### 稳定执行依赖 Agent Harness

模型负责开放式决策，Harness 负责确定性约束。MobileUse 分层反思架构（Proactive Exploration + Hierarchical Reflection：previous progress / updated progress、Rule-based 判定（repeat? action cycle?）、task trajectory summary、global reflection、Finish? 判定）。

> Harness 必要，但不能盲目堆叠框架。构建**轻量、可观测、可消融、与模型匹配**的 agent harness。

### 构建异常状态感知能力

| 异常类型 | 表现 | 处置 |
|---------|------|------|
| **环境异常** | 位置权限、用户协议、支付确认等安全合规风险操作 | 主动识别拦截 → 用户接管 |
| **用户意图异常** | 用户指令模糊、执行过程多选项决策带来的执行不确定 | 追问澄清 |
| **模型异常** | Agent 对能力边界的认知缺失导致的动作不匹配、决策失真以及循环执行错误动作 | 状态感知：使智能体能够识别"看不清、想不准、做不稳"的状态 |

### 构建主动的人机交互范式

针对当前 Agent 交互不透明、低效导致的用户体验缺失，开发基于可视化信息主动向用户反馈当前执行状态和任务进度的主动交互系统；对模糊意图澄清进行主动追问，降低用户与 agent 交互的成本。幻灯片演示了 Cost/Reward 权衡式追问策略（如 "I don't know." [Cost 3] + [Reward 0]；"I prefer option A." [Cost 4] + [Reward 1]；ask_question / generate_ui 等动作），并在 SWE-bench / TravelGym / T2-bench / WebArena 等基准环境中验证。

### 构建个性化、持续学习的智能体

面向用户在长期 Agent 使用过程中形成的稳定偏好、重复行为和场景化习惯，基于已有或在建的用户记忆系统，建立记忆增强的个性化执行，实现对用户历史操作轨迹、任务意图等信息的存储与调用，共建记忆-执行链路的协同进化能力。四类沉淀：

- **Agent**：训练数据的沉淀
- **Memory**：用户上下文的沉淀
- **Self-evolving skills**：标准工作流的沉淀
- **Harness**：适配真实环境的经验沉淀

四个演进场景：**1.1 GenUI 冷启动**（没有足够历史信息时，先根据当前任务生成可操作界面，把品牌、商品、规格和确认步骤组织成用户能直接选择的点单流）；**1.2 模糊意图匹配**（用户说"不太甜""下午别影响睡觉""想喝咖啡味"这类模糊需求后，匹配到甜度、咖啡因、口味和饮用场景等可执行属性）；**1.3 个性化信息学习**（从用户接受推荐、修改规格、跳过候选和补充说明中持续更新 Preference Memory，让下一轮筛选更贴近个人习惯）；**1.4 模糊点单推荐演变**（偏好越来越明确时，界面从泛化候选逐步演变为个性化推荐：过滤不合适选项、解释推荐依据，并把最可能满足需求的商品前置）。

## 一次任务，驱动完整闭环

真实任务示例（美团外卖点单「生椰拿铁控糖版（两杯起购）」轨迹，时间为每步动作耗时）：

- Step 19 (1.09s)：action: click——点击"+2份起购"按钮
- Step 20 (1.16s)：action: click——点击"加入购物车"按钮
- Step 22 (1.25s)：action: click——点击弹窗关闭按钮
- Step 23 (0.02s)：action: complete——任务完成

（底部结算栏显示"到手约 ¥21.8 … 免配送费 / 去结算"，件数等细节无法辨认。）

## 环境标准化：让每次结果都可信

- **代码环境**是标准的 sandbox——可重置、可回滚、可验证；**GUI 环境**是动态变化的——缓存、个性化推荐的记忆、结果难验证
- **AndroidWorld 在 3 个任务参数种子上的成功率为 27.6%、26.3%、33.2%；即使模型与框架不变，任务参数也会显著改变结论**
- **ColorBench**：图结构框架评估复杂长程任务，支持动态执行 / 稳定重复 / 多路执行

## 结论

**GUI AGENT ENGINEERING：把每次执行变成下一轮能力**——真实环境可控、数据飞轮迭代、框架持续进化。

## 参考文献（幻灯片引用）

1. Liu, Mingjie, et al. "ProRL: Prolonged reinforcement learning expands reasoning boundaries in large language models." NeurIPS 38 (2026): 17998-18031.
2. Yao, Shunyu, et al. "z-bench: A benchmark for Tool-Agent-User interaction in real-world domains." ICLR 2025.（首字母花体，无法可靠辨认）
3. Li, Yang, et al. "GUI-CEval: A hierarchical and comprehensive chinese benchmark for mobile gui agents." CVPR 2026.
4. Li, Ning, et al. "ColorAgent: Building A Robust, Personalized, and Interactive OS Agent." arXiv:2510.19386 (2025).
5. Zheng, Congmin, et al. "Adaptive Milestone Reward for GUI Agents." arXiv:2602.11524 (2026).
6. Li, Ning, et al. "Mobileuse: A hierarchical reflection-driven GUI agent for autonomous mobile operation." NeurIPS 38 (2026): 40361-40388.
7. Ma, Xinbei, et al. "Retrospective progress-aware self-refinement for llm agent training." arXiv:2606.14302 (2026).
8. Ma, Xinbei, et al. "Communication Policy Evolution for Proactive LLM Agents." arXiv:2606.14314 (2026).
9. Wu, Zheng, et al. "Verios: Query-driven proactive human-agent-gui interaction for trustworthy os agents." arXiv:2509.07553 (2025).
10. Rawles, Chris, et al. "Androidworld: A dynamic benchmarking environment for autonomous agents." ICLR 2025.
11. Song, Yuanyi, et al. "Colorbench: Benchmarking mobile agents with graph-structured framework for complex long-horizon tasks." Proceedings of the ACM Web Conference 2026.

## 相关页面

- [[agent-harness-overview]] — Harness 综述：六承重层（主循环/工具/记忆/状态/权限/验证）
- [[harness-engineering-evolution]] — Harness 工程演进与企业落地（八大工程难题）
- [[distributed-agent-hosting]] — 腾讯云 Agent Runtime：分布式托管基础设施
- [[loop-engineering]] — Loop 工程：数据飞轮与持续迭代的对照
- [[harness-engineering-practice]] — Harness 工程实践（Human-first → Agent-aware）
- [[harness-as-backend]] — Harness 作为新后端
- [[agent-hook-governance]] — Agent Hook 治理：执行层确定性约束
- [[workbuddy-agent-product-design]] — WorkBuddy 五层 Harness：前馈+反馈控制
