---
title: 通义 Mobile-Agent 多模态 GUI 智能体家族
description: 通义实验室（阿里云）Mobile-Agent 系列——多模态、多端 GUI 智能体家族（V1/V2/E/PC-Agent/v3/GUI-Owl/v3.5）与 GUI 基础模型的技术演进、数据合成链路与 Agentic RL
aliases: [Mobile-Agent, mobile-agent-gui-family, 通义GUI智能体, GUI-Owl, GUI智能体, Mobile-Agent-V3.5]
tags: [ai-agent, concept, tool]
sources: [2026/08/11/Foundation Model 2.0论坛/01-徐海洋-通义多模态、多端GUI智能体_Mobile-Agent.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# 通义 Mobile-Agent 多模态 GUI 智能体家族

> 来源：2026 Agent 大会「Foundation Model 2.0 论坛」演讲《多模态、多端GUI智能体 Mobile-Agent》，徐海洋（通义实验室算法科学家）。

## 背景：GUI Agent 是重要技术趋势

- 大模型智能体：观察环境并作出行动以达致目标的自主实体（引 Lilian Weng blog、Wang et al. Survey）。
- 相比传统基于 RL 的智能体（数据采样专有环境和低效、面向特定任务、稀疏奖励和长时段问题），大模型智能体的优势是：丰富的世界知识、推理/规划能力、工具使用（检索、code 等）、In-context Learning。
- 两类近期 AI Agent 应用：
  - **GUI Agent**——[硬]"眼睛"&"手"，环境感知和行动执行，[自动化]操作主导型，适用简单重复任务（生活娱乐、自动化办公），示例：Operator、Apple Intelligence、Mobile-Agent、豆包手机。
  - **Code/CLI Agent**——[软]"大脑"，思考规划和综合分析，[智能化]知识密集型，适用复杂创作任务（办公生产力场景），示例：Code（Claude、OpenAI、Gemini、Qwen）、GUI+CLI 桌面端 Agent（CodeX、Claude Code、Qwen Code）。
- "围绕 Mobile、PC、Web 的 GUI-Agent 是未来的重要技术趋势之一，替代人类操作、提升生产效率"，示例包括 Claude (Computer)、Apple Intelligence、OpenAI Operator、豆包手机、AutoGLM。
- **GUI+CLI 桌面端 Agent**：基于 GUI 的操作——像真人一样通过实时截图"看到"屏幕内容，再模拟鼠标键盘操作。示例：Codex 调 Adobe Firefly 做 AI 视频生成、打开 Photoshop 做播客封面设计；官方演示中一个指令即自动启动应用、复现 bug、修复并测试。
- **Hybrid-Agent**（基于 Qwen3.7-Plus）：将大模型代码生成能力与 GUI 自动化执行深度融合，实现从需求分析到版本迭代的 APP 全链路开发。Agent 持续稳定运行 **11+ 小时**，完成一款英语单词学习 APP 的完整研发闭环；累计生成代码超过 **10,000+ 行**，触发 Agent 调用超过 **1,000+ 次**。

## Mobile-Agent 家族与多端覆盖

Mobile-Agent：强大的 GUI 智能体家族（CCL2024、CCL 2025 Highlight System；Github 8.9k stars）：

| 版本 | 定位 |
|------|------|
| Mobile-Agent-v1 | 单智能体多模态手机操作 |
| Mobile-Agent-v2 | 多智能体多模态手机操作 |
| Mobile-Agent-E | 多智能体、自我进化手机操作 |
| PC-Agent | 多智能体多模态 PC 操作 |
| GUI-Owl & Mobile-Agent-v3 | 多模态、多平台 GUI Agent |

覆盖端：手机、平板、车机、PC & Web。

**核心挑战**（演讲归纳为四类）：
- **UI 界面感知**：UI 元素多样，不同系统、机型、APP 差异大，需要精确定位精确操作
- **长程任务规划反思**：需要复杂任务规划、最优路径选择、多图理解推理能力
- **Foundation Agent 能力**：工具 / Memory / Agent 框架 / Critic / CLI 等能力
- **Long Horizon 路径选择能力**：Long Horizon 最优路径选择路由能力

## 代际演进：V1 → V2 → E

- **V1（观察-思考-行动）**：解决大模型缺乏输出精确坐标的 grounding 能力——屏幕文本定位用 OCR 工具检测识别文本框、图标定位用图标分割检测工具；行为空间含点击文本、点击图标、打字、上划&下划、返回上一页面、返回桌面、结束共 7 类。
- **V2（多智能体）**：冗长且图文交错格式的操作历史会大大增加智能体追踪任务进度的难度；采用 Planning Agent、Decision Agent、Reflection Agent 三智能体框架（规划/决策/反思三阶段，含 Focus Content Navigation 与 Task Progress Navigation）。图中对比显示多智能体架构相比单智能体准确率大幅提升（图中数据：50.7 vs 86.5）。
- **Mobile-Agent-E（自主进化）**：解决复杂任务——层级智能体（Manager 规划、Operator 执行、Action Reflector、Notetaker）+ 两个经验 Experience Reflectors 根据当前任务的操作记录和错误日志，对 **Tips 和 Shortcuts** 进行可能的优化和更新，沉淀进长期记忆（Long-term Memory）。典型场景：多平台比价任务（Amazon/Walmart/Best Buy 找最便宜并停在加购页）。

## Foundation Agent for GUI

- **Qwen2.5-VL（认识世界到理解世界）**：认知现实物体能力、GUI 交互能力（基于截屏执行 click、type、home 等动作）、文本理解、Grounding（生成 bounding boxes 或 points 准确定位物体、输出稳定 JSON）、OCR 能力。
- **Mobile-Agent-v3 & GUI-Owl**：
  - 大规模环境基础设施：云端跨平台虚拟环境（ECS 集群、Android 模拟器、Windows/Linux 模拟器、Web 服务器；轨迹存储用 OSS、模型调用走百炼、SFT/RL 训练在 PAI GPU 集群）。
  - 自进化 GUI 轨迹生产框架；Offline Hint-Guided Rejection Sampling、多智能体框架蒸馏、迭代在线 Rejection Sampling。
  - 可扩展环境 RL：解耦的 rollout-update 框架，支持并行运行。
  - 在 OSWorld、OSWorld-G、Android World、ScreenSpotPro、Android Control、MMBench-GUI L1 (Hard) 等多个 GUI 基准上为开源新 SOTA（图中数据，如 OSWorld 43.9、MMBench-GUI L1 Hard 94.2；模型与数值的逐项配对部分无法逐字确认）。
  - **数据合成链路**：① Grounding 数据（开源 Grounding 数据集 + A11y tree 合成 + SAM 稠密标注 + 数据精炼 + 细粒度字/字符 Grounding）；② 自主进化轨迹合成（在线虚拟环境 + Query 生成 + 轨迹生成 + 正确性判定模块 + Query 特定引导生成）。
  - **Agentic RL 能力提升**：异步 Rollout 服务（Scalable Experience Maker）；在线过滤 + 经验管理（TRPO）优于在线过滤（DAPO）与离线过滤（GRPO）。
  - **Mobile-Agent-V3 Agent 框架**：Manager Agent（任务拆解）+ RAG Module（知识检索）+ Worker Agent（执行）+ Reflector Agent（反思）+ Notetaker Agent（记录）。
- **Qwen3-VL（明察、深思、广行）**：像人一样操作手机和电脑（打开应用、点击按钮、填写信息）；带图推理（结合工具做细粒度识别与逻辑分析）；手绘草图转网页代码。
- **Mobile-Agent-v3.5 & GUI-Owl-1.5**：
  - 多平台协同（PC、浏览器、手机统一控制）；长短记忆与工具（Tool User Integration、API 调用）；多 Agent 协同与任务分解；跨平台统一 RL（解耦 rollout-update）。
  - "SOTA GUI-Agent Model across 20+ GUI benchmarks"（涉及 OSWorld-G、OSWorld-Verified、ScreenSpot-Pro、VisualWebArena、WindowsAgentArena、AndroidWorld、MobileWorld、GUI Knowledge Bench 等，具体数值为图表数据）。
  - Action Space：Computer Use（left_click、double_click、triple_click、scroll、type）、Browser Use（click、goback、interact、wait、type）、Mobile Use（answer、long_press、system_button、click、terminate）；扩展工具：Amap MCP Tool、Office Tool、PyCharm Tool；支持压缩历史（Compressed Histories）与多轮对话。
  - 数据合成：硬 Grounding 数据（多窗口高分辨率、不可行数据生成）、Unified CoT 合成（观察-反思-记忆-世界建模-知识注入）、多智能体知识库增强。
  - RL 训练：Multi-Platform Rollouter + Rollout-Trainer Aligner + Interleaved Trainer；正负样本混合的扩展 Rollout 缓冲（增加多样性、减少丢弃、保持分布）。

## 开源应用

- **Hugging Face / 魔搭 ModelScope**：GUI-Owl-1.5 系列模型（mPLUG/GUI-Owl-1.5-8B-Think、32B-Instruct、4B-Instruct、2B-Instruct 等，2026-02-26 更新，含 GGUF/MNN/MLX 社区量化版本）。
- **阿里云百炼**：模型 `gui-plus-2026-02-26`（GUI-Plus 系列，支持思考模式与非思考模式，跨平台多 APP 任务理解与执行大幅提升，支持工具调用）；页面截图显示价格约为输入 **1.5 元/每百万 tokens**、输出 **4.5 元/每百万 tokens**（截图含单位切换控件，数值以百炼页面为准）；配套无影 AI 云手机与 MobileAgentTest 在线体验空间。
- **GitHub**：https://github.com/X-PLUG/MobileAgent（17 位贡献者、573 forks、README 标记 #5 Repository Of The Day）。

## GUI 工具调用 & Benchmark

- **OSWorld-MCP**：158 个高质量 MCP 工具；研究对比 GUI 式操作 vs MCP 工具调用两种任务执行方式。
- **Tool-CUA（GUI-Tool 最优路径选择）**：纯原子 GUI 动作（CLICK/TYPE/SCROLL）"刚硬且脆弱"，GUI-Tool 交错动作"灵活且强大"；ToolCUA-8B 在 OSWorld-MCP（feasible）上 48.4，高于 EvoCUA-32B 46.8、Gemini-3.1-Pro 41.1、Claude-4.5-sonnet 40.8（图中数据）；方法含 Tool-Bootstrapped GUI RFT（Warmup SFT → 单轮 GRPO 关键步骤 RL → GUI-MCP 环境在线 Agentic RL）。

## Qwen3.5 / 3.6 / 3.7 视觉 Agent

- **Qwen3.5**：视觉智能体，自主操作手机与电脑（移动端适配更多主流应用、自然语言指令驱动；PC 端跨应用数据处理、多步骤流程自动化）。演示任务：在 YouTube 搜索 qwen3vl、点赞、稍后观看、评论。
- **Qwen for Chrome 浏览器插件**：浏览器侧边栏与 Qwen 对话，Agent 模式下感知网页内容、规划操作步骤，以 BrowserAgent 形式执行点击、输入、跳转、验证；演示覆盖云服务器采购→实例扩容→运维升级的完整链路。

## 未来展望（技术角度）

- Agentic RL Scaling，提升自主推理和知识进化
- CLI+GUI 相结合桌面端 Agent，自主解决问题、完成生产力任务
- 个性化交互与记忆

## 相关页面

- [[agent-mcp-protocol]] — OSWorld-MCP、Tool-CUA 与 MCP 工具调用生态
- [[kimi-long-horizon-agentic]] — 长程 Agentic 任务（同为论坛演讲，长程任务基准与案例）
- [[agent-evaluation-framework]] — GUI 多基准测评与 Agent 评估维度
- [[agent-multi-agent-collaboration]] — Mobile-Agent V2/E 的多智能体框架
- [[agent-memory-system]] — Mobile-Agent-E 的 Tips/Shortcuts 长期记忆
- [[agent-autonomous-planning]] — Manager/Worker 层级规划
- [[llm-cost-optimization]] — 大模型成本控制（Hybrid-Agent 长时运行的 Token 成本话题）
- [[glm5-model]] — 同期竞品大模型系列
