---
title: Memory Scaling 与 LycheeMemory——大模型持续学习机制探
description: 哈工大（深圳）李东方演讲：记忆智能背景与 Memory Scaling 定义/定律、10 项记忆前沿工作综述（MemoryArena/MemoryBench/MemTrace/ATM-Bench/MemFail/SubtleMemory/RecMem/MemPro/Neural Procedural Memory/Mem-π）、LycheeMemory 三层认知记忆架构与构建/检索/自进化机制、LycheeMemory-7B 模型、G-RA/AAPO/DePO-REAL 强化学习、LoCoMo/LongMemEval-S/PinchBench 测评数据、华为 openPangu 落地与 DeepSeek-V4 国产算力后训练
aliases: [Memory Scaling, LycheeMemory, LycheeMemory-7B, 持续学习, 记忆智能, 大模型记忆]
tags: [ai-agent, concept, tool]
sources: [2026/08/11/Agent记忆工程论坛/02-李东方-Memory Scaling：大模型持续学习机制探.pdf]
created: 2026-08-11
updated: 2026-08-11
---

# Memory Scaling 与 LycheeMemory

**演讲**：李东方，哈尔滨工业大学（深圳）（2026-07-25；2026 Agent 大会「Agent记忆工程论坛」，DataFun AGENTIC AI 超级智能体系统架构峰会，44 页）

目录：01 大模型记忆智能；02 智能体记忆工作；03 相关工作与展望。

## 01 大模型记忆智能

### 背景

- **大模型需要持续学习**：开放世界环境是动态和演进的，正不断涌现新概念、新现象；大模型必须克服其静态与固化局限，实现与环境的同步演化。引用人工智能先驱、图灵奖获得者 Allen Newell 教授、司马贺教授：智能是"实现自身目的并适应环境"。
- **记忆是人类持续学习的基石**：人类智能中，记忆是动态的生成与重构的过程，对环境信息进行深入抽象、关联与整合，是持续、连贯和个性化交互等高级认知的核心基石。记忆三个环节：
  - **编码**：外部信息重组、转化。学习效率取决于记忆编码策略，多通道编码、情景联想等策略显著提升学习效果
  - **组织**：分层分类保存长期记忆，保留所学内容使后续学习更高效
  - **检索**：从长期记忆中提取信息，巩固记忆存储、激发元认知能力，促进推理、抽象与联想
  - 引用文献：Pyc & Rawson (2010, Science)、Bjork (1994)、Karpicke & Roediger III (2008, Science)
- **记忆智能是实现 ASI 的必由之路**：五级智能阶段图——L1 特定任务的文本智能（特定文本任务的模型）→ L2 多任务通用语言智能（通用大语言模型）→ L3 多数据类型跨模态智能（以语言为核心的多模态大模型）→ L4 具备任务执行能力的通用智能（通用智能体）→ L5 具备持续学习能力的记忆智能（记忆智能）；对应三个阶段：语言智能（能说、能写、能理解）→ 多模态与智能体能力（能感知、能规划、能执行）→ 记忆智能（能积累、能记忆、能学习）。"当前大模型正由通用智能体阶段，逐步迈向具备持续学习能力的记忆智能。"

### Memory Scaling 定义

通过持续沉淀知识、对话、反馈、执行轨迹与上下文经验，使得智能体在不频繁更新大模型权重的情况下，持续学习，不断提升准确率、效率与个性化。

**Memory Scaling Law**：当有效记忆规模持续扩大时，长期任务中的错误率、重复推理成本和上下文遗漏会呈现可预测下降。图示：无显式记忆时错误率很快进入平台期；朴素记忆/普通 RAG 先下降、后受噪声与冲突拖累；可治理记忆（选择性写入 + 证据召回 + 合并/清除）随有效记忆预算/规模 M 扩大持续下降。注意 M 指"可检索、可更新、可治理的记忆"，不是原始日志量。

**核心主张**：智能的持续提升，不只来自参数和上下文变长，也来自有效记忆规模的可控扩展和治理，包括写入、维护、检索、遗忘等。

### 前沿工作综述（10 项）

| 工作 | 发表 | 解决的问题与要点 |
|------|------|------------------|
| **MemoryArena** | ICML 2026 | 记忆评测停留在事实回忆 → 构造连续行动任务（4 类环境：捆绑购物、多人旅行规划、逐步网页搜索、数学与物理推理），检验记忆能否改善后续决策与任务完成。任务链变长时长上下文、检索增强与外部记忆都更容易失效；外部记忆有时能帮助搜索和推理，但通常带来更高延迟 |
| **MemoryBench** | ICML 2026 | 用户纠正难以沉淀为长期能力 → 模拟"反馈→再执行"闭环（User Simulator→Task Provider→Score Computation→LLM-as-judge 可选）。不只检查有没有保存聊天记录，而是看用户纠正之后下一次会不会真的做得更好；复杂记忆系统未必稳定优于简单检索，构建时间也可能显著增加 |
| **MemTrace** | arXiv 2026 | 总分掩盖错误来源 → 用重复追问、时间变化和证据条件拆解维护、检索与使用失败。问题类型拆成 current/past/change；证据条件区分 present/missing/conflict；总分相同可能来自完全不同的错误路径，评分差距可达 10 倍 |
| **ATM-Bench** | arXiv 2026 | 真实个人记忆不只有聊天记录 → 数据来自一个人近四年的照片、视频和邮件，共 1,038 个带证据的问题；挑战含个人化指代（Needle in a Haystack）、时间更新与多证据整合（Outdated Evidence Trap）、位置感知（Invisible Link）。当前记忆系统在困难集上都不到 20%（A-Mem 15.0、Mem0Agentic 16.5、Self-RAG 16.1）；即使直接给出正确证据，成绩也只有 47.3，说明检索与证据整合本身也是瓶颈 |
| **MemFail** | arXiv 2026 | 端到端分数难以定位 → 把记忆系统拆成总结、存储、检索环节定位错误发生在哪一步；五组数据分别检查条件丢失、事实覆盖、误导检索、长链检索等；比较 Mem0、SimpleMem、A-MEM、StructMem 并改变检索条数 k 和所用模型；多放内容有时补回摘要信息、有时增加噪声；换更强模型也不一定有效 |
| **SubtleMemory** | arXiv 2026 | 多条记忆之间的关系容易丢失 → 构造互补（互补兼容）、细微差异（可细微区分）、冲突（互斥）三类语义变体，评估系统能否保持关系语义。数据含 1,522 个问题、10 条长历史、1,090 组关系可控的变体；GPT-5.4 上最佳独立系统 A-Mem 达 70.0%，完整记忆上限为 85.4%（图中另一成绩 68.7% 对应条件不确定） |
| **RecMem** | Findings of ACL 2026 | 每轮对话都总结导致成本高、噪声早写入 → 只在模式反复出现后固化记忆：先把交互放进轻量缓冲区，同类内容反复出现、形成稳定模式后才调用大模型整理（Recurrence-based consolidation vs Eager consolidation）。LoCoMo（GPT-4.1-mini）上准确率 **81.10**，构建记忆 **19.32 万 token（193.2K）**，较 Mem0/A-Mem 减少约 **87%**（对比：MemoryOS 400.7K、A-Mem 1459.9K、Mem0 1520.8K） |
| **MemPro** | arXiv 2026 | 让整套记忆程序自我进化 → 把整理、保存、检索、回答整套程序都作为优化对象；每个版本包含提示词、可运行代码和评测记录，保留多条版本分支（MCR Version Tree）。覆盖 LongMemEval、LoCoMo、HotpotQA、NarrativeQA；只迭代 5 次就超过所有非 MemPro 基线；同时改代码明显好于只改提示词 |
| **Neural Procedural Memory** | arXiv 2026 | 程序性经验难以完全写成文字 → 把成功与失败行动轨迹对比学习总体策略和局部纠错，经验保存为方向向量、执行新任务时注入模型，无需重新训练参数。Qwen3-8B 无记忆平均分 **30.63**，使用神经记忆后 **36.32**，与文字流程结合达 **41.89**，ALFWorld 上达 **66.42** |
| **Mem-π** | arXiv 2026 | 相似旧记录不一定有用 → 训练记忆模型按当前任务生成或放弃指导（d=[ABSTAIN] 或 d=[GENERATE]@m），记忆从检索片段转向临场建议（Decision-Content Decoupled Policy Optimization）。四类智能体任务上平均相对提升约 **20%**，WebArena 接近 50%：WebArena 35.0→43.1、WorkArena 42.0→50.3、ALFWorld 达 91.6 |

**趋势总结**：长期记忆研究正在从"保存更多信息"转向"让记忆被正确构建、诊断、更新、优化并真正改善行动"——评测从回忆走向行动（MemoryArena/MemoryBench/ATM-Bench）、诊断从总分走向失败定位（MemTrace/MemFail/SubtleMemory）、方法从全量写入走向选择性固化（RecMem）、记忆从文本记录走向可优化程序（MemPro/Neural Procedural Memory/Mem-π）。下一阶段关键问题不是把记忆库做大，而是让记忆系统知道该记什么、何时用、怎么用错了还能改。

### 我们的工作定位

构建以记忆为核心的智能体长效学习机制，依托可检索、可更新、可复用记忆能力拓展认知边界，形成越用越强、自主迭代、高效协同的大模型智能体系统。时间线：2020-2023 Training Scaling → 2024-2025 Test-time Scaling → 2026- Memory Scaling。相关工作地图：参数化记忆（LycheeMemClaw）、非参数记忆（LycheeSEEM）、智能体记忆/上下文记忆（LycheeDecode、LycheeCluster）、记忆形成（强化学习 DePO、AAPO）、记忆管理（记忆选择 ComMCS、G-RA、记忆组织）。

## 02 智能体记忆工作

### 痛点

Agent 进入复杂任务与真实工作流的基础性短板：缺乏稳定、准确且低成本的长期记忆能力。当前 Agent 记忆方案：简单的上下文堆叠（记不住）、粗粒度的向量检索（找不准）、高昂成本换取有限精度（用不对、成本高）。

### LycheeMemory 系统

面向真实智能体场景开源的大模型记忆管理基础设施（LycheeMemory: Lightweight Long-Term Memory for LLM Agents，GitHub 开源，pip install lycheemem，lycheemem-cli）：
- 支持本地启动 API 服务；PIP 安装，提供记忆可视化
- 提供原生 OpenClaw 插件；支持 MCP、Hermes 接入
- 支持离线运行与本地数据存储

**总体架构**：3 层认知记忆架构，融合 7 种记忆类型评分，实现 5 类记忆推理智能体：
- **Working Memory**：管理活跃上下文
- **Semantic Memory**：7 种语义类别——fact（事实）、preference（偏好）、event（事件）、constraint（约束）、procedure（流程）、failure-pattern（失败模式）、tool-affordance（工具能力）
- **Procedural Memory**：保存可复用的操作方法知识（Doc-markdown：完整 Markdown 文档，描述步骤、命令、参数和注意事项；技能条目）
- 存储：向量（用于相似度搜索）+ 元数据（使用计数、最后使用时间戳、前置条件）+ SQLite（FTS5 全文检索）+ LanceDB（向量索引）

**记忆构建**：语义片段级固化（基于语义意外度与片段内聚度识别事件边界，仅在语义转折处进行记忆固化，减少大模型调用次数）；上下文独立表示（对片段中事实、偏好、事件与流程进行抽取，完成指代消解与时间归一化，使记忆脱离原始对话后仍可独立理解）；跨片段连续性（通过消歧反馈传递实体、别名与指代关系，在不重新读取完整历史的情况下保持记忆一致性）。

**记忆检索**：结构化记忆组织（基于每条记忆的实体、关键词/主题自动建立多维索引：实体、主题、时间、会话的独立索引 + 实体×主题交叉复合索引）；基于检索规划的多路召回（将用户查询转化为多组实体、主题、时间、事件类型等线索，指导不同召回通道协同工作，兼顾查询效率、信息完整性与可解释性）。

**方法优势**（相比训练式路由、多 Agent 管理和逐轮固化方法，用更少的记忆操作构建更可复用、更可验证的长期记忆）：无需训练（不依赖 RL、多 Agent 学习或小模型微调，现成 LLM + Embedding + 轻量索引即可部署）；高效轻量（仅在语义片段边界触发记忆固化，避免逐轮 eager consolidation，无需多 Agent 协作、反复 Judge、复杂记忆更新）；连续可靠（保持跨片段实体与指代连续，最小 Memory 单元保留实体、主题、时间与类型，可回溯原始片段，保证证据链充分可靠）。流水线：语义片段触发 → 原子记忆编码 → 消歧状态传递 → 多维索引检索。

**记忆检索案例（LoCoMo）**：
- Case 1 Multi-Hop："How do Jon and Gina both like to destress?" Router 拆成 Gina Destress / Jon Destress 两条证据路径，分别检索 FACT（Gina uses dance…Session 1）与 PREFERENCE（Jon loves dance…Session 11），模型回答 By dancing。
- Case 2 跨会话聚合："What percentage of packed shoes did I wear on my last trip?" Router 1: Packed Shoes（FACT: user packed 5 pairs of shoes…Session 38）、Router 2: Worn Shoes（RAW USER TURN: …only wearing two - my sneakers and sandals…Session 4），计算 2/5 × 100% = 40%。
- 检索阶段 Planner 将用户问题转化为可验证的证据路径，再检索相关记忆片段。

**记忆自进化**：从自身运行轨迹中自学习更合适的记忆策略，使长期记忆随真实任务场景持续适配（记忆构建过程：语义切分、多元组织；记忆检索过程：多路检索、验证进化 → 记忆行为自进化）。

**记忆模型 LycheeMemory-7B**：长短记忆协同的大模型，完成记忆构建、更新、推理等，将原始交互流转化为记忆单元，实现百万级长文推理的高准确率，已开源 Huggingface。压缩记忆与动态推理：长文推理能力（几乎）无损外推至 **1.75M Token**；推理速度是 MemAgent 的 **6 倍**、全上下文的 **10 倍**。论文：Dynamic Long Context Reasoning over Compressed Memory via End-to-End Reinforcement Learning, ACL 2026。

**高效强化学习（一）**：针对长程交互训练中方差高、不稳定的问题，提出门控奖励集成 **G-RA** 和多轮记忆生成策略优化 **AAPO** 方法，平衡过程奖励和长程奖励，最终提升推理效果和训练稳定性（多粒度奖励驱动的深度强化学习 + Action-Level 重要性权重聚合；奖励未对齐导致长程奖励忽略的问题得到大幅缓解）。论文：Improving Value-based Process Verifier via Low-Cost Variance Reduction, AAAI 2026。

**高效强化学习（二）DePO-REAL**：针对异步 RL 策略分布漂移与训练精度跷跷板等问题，提出双轨去噪校准强化学习框架，显著提升训练效率、稳定提高模型精度，并增强多种任务能力。输入（任务/数据/奖励）→ REAL（动态优势估计的策略校正）/ DePO（异步训练去噪策略优化：采样权重噪声、错误优势基线）→ 策略梯度优化 → 输出（多能力高精度基座模型）。对比（AIME2025、LiveCodeBench、Arena Hard，训练步数/准确率）：基线 300/27.7、SOTA 260/32.9、DePo-REAL **100/34.1**——减少训练步数 **60% 以上**、大幅提升模型精度，超越现阶段多任务 RL 学习 SOTA 指标；模块化设计，代码少于 200 行，依赖少、可复现。论文：Mitigating Policy Entropy Collapse via Denoised Policy Optimization, Under Review。

### 记忆测评

**LoCoMo 测评**（表格）：

| 方法 | 准确率（%） | 构建 Token (K) | 查询 Token (K) |
|------|-----------|---------------|---------------|
| MemoryOS | 67.6 | 400.7 | 5.04 |
| A-Mem | 68.8 | 1459.9 | 5.56 |
| Mem0 | 62.9 | 1520.8 | 2.11 |
| **LycheeMemory** | **89.2** | **204.1** | 4.01 |

在显著降低记忆构建 Token 消耗的同时，在 LoCoMo 和 LongMemEval-S 上取得更高的回答准确率（LongMemEval-S 数值图中未标注）。

**PinchBench 测评**：接入 LycheeMemory 的 OpenClaw 实现更优越的性能——整体评分提升 **6%**；Token 消耗平均下降 **71.3%**；总使用成本降低 **54.8%**；Hard 最高分数 **91.2%**。重点不只是"能记住"，而是在有限 Token 预算下"找得准、用得上"；每投入 1 元记忆检索成本，最高可节省约 54.8 元大模型推理成本，**ROI 达 5480%**。

### 记忆插件与场景落地

- **多平台插件接入**：支持 OpenClaw、Claude Code 和 Hermes 等 Agent Runtime，通过插件将长期记忆能力接入真实交互流程
- **smart_search 统一封装**：默认调用 smart_search，将多层检索、重排、压缩与上下文构造封装为统一记忆召回接口
- **Hook 生命周期管理**：通过会话开始、消息追加、会话结束等 hook，自动完成对话写入、记忆巩固与状态同步，减少手动调用成本
- **华为合作**：通过华为 OP 专项合作，记忆方法成功应用到 Pangu-Embedded-7B；构建了高信息密度与叙事连贯性的情节记忆，完全国产算力落地于 openPangu 模型，实现超越复杂基线方案的长程深度推理，期待在 openPangu 2.0 进一步应用（Structured Episodic Event Memory, ACL 2026）
- **开源 OpenClaw Skill**：无缝接入智能体工作流，提供即插即用的结构化记忆技能（情节事件帧、图记忆层、记忆对象样例；项目主页 https://ryantoleco.github.io/seem-skill/）

## 03 相关工作

- **DeepSeek-V4-Pro 官方发布**：官方媒体发布与中央部委关注，全网总阅读量超 **500 万**、总转发量超 **5 万**（新华社、人民网、大公报、文汇报、深圳特区报、深视新闻、南华早报、深圳发布、证券时报、凤凰网等）。河套学院公众号首发：1.6 万亿参数 MoE，昇腾 910C 集群，承接超大参数模型训练链路关键环节；新华社视频报道聚焦"1.6 万亿参数大模型后训练，国产算力跑通关键一环"。
- **SLAI T-Rex（全球首发）**：基于国产算力集群完成 DeepSeek-V4-Pro 全参数后训练工程实践，实现 V4 Flash/Pro 的 SFT/续训练（V4-Flash mindspeed 续训、V4-Flash torchtitan 续训、V4-Flash 8K SFT 训练、V4-Pro torchtitan 续训、V4-Pro 8K SFT 训练均已跑通）；未来将依托国产算力底座，打造昇腾亲和的新一代记忆原生的模型架构。技术报告《SLAI T-Rex: Full-Parameter Post-training of the DeepSeek-V4 Family on Ascend SuperPOD》（Shenzhen Loop Area Institute）：MFU 从 **11.67% 提升至 34.22%**（相对开源基线配方 **2.93x**）；面向复杂运筹（OR）建模的 CPT-SFT 数据管线，10K 高质量 SFT 样本，zero-shot Pass@1 达 **71.81%**，超越 GPT-5.4-Mini 3.98 个百分点、超越基础 DeepSeek-V4-Flash 11.27 个百分点（对比图另见：Gemini-3-Flash 66.82、DeepSeek-V4-Flash 60.54、GLM-5.2 56.60，其余条目 OCR 不确定）。另一页 MFU 提升趋势图：社区基线（TorchTitan 等开源配方）起步，经并行策略优化、硬件亲和优化、算子优化逐级提升至约 33.95%，整体提升约 2.91X（各阶段数值部分标注不清）。训练数据集续训数据 2.6T→50B、专家构建数据，进行中；测评集初步筛选持续构造中（ORGEval 394、NL4OPT 245、OptMath 166、OptiBench 605 等，部分条目 OCR 不确定）。
- **ITU 全球电信 AI Agent 挑战赛**：AI 训练平台学生团队获世界冠军并受邀报告——SLAI BOYS 位列 Telco Troubleshooting Agentic Challenge（ITU，€40,000 EUR，1394 人加入、354 活跃，2026-04-17 至 06-03）Track B Phase 3 Final 第一，与 Cisco_Team 等专业队伍同场竞争；团队成员为深圳河套学院（SLAI）博士生（Team Lead: Lu Zhengxuan，AI4SE；Liu Jiajun、Jin Wenlong、Yin Zihao 等），国产算力支持；ITU AI for Good 平台推动，联合 GSMA、ETSI 等国际组织。

### 未来工作与展望

- 进一步提高对记忆机理的认知，不仅是工程实现，挖掘关键科学问题；从面向 Agent 的长期记忆系统设计，到记忆原生大模型新型架构的构建
- 方向一：面向无限长序列的记忆原生的**动态图索引新型架构**（降低长文本推理开销，提升大模型处理效率；物理空间线性时间线 → 语义投影/相似度 → 语义空间拓扑/图结构，高相似度活跃边，层级记忆层 Memory Tree）
- 方向二：**行动感知驱动的智能体层次化记忆**（让记忆真正参与写入、检索、巩固和注入全周期）

**Memory Scaling 研究框架展望**：记忆写入机制（何时写入：触发条件与写入策略；粒度选择与重要性评估；增量/批量/流式写入）；记忆结构化（数据组织：层级/图/表结构；本体与模式：知识图谱/Schema/标签体系；关联建模：实体对齐与关系建模）；记忆压缩与抽象（压缩方法；抽象层次：事实→事件→模式→原则；信息保留）；记忆更新与冲突处理（更新机制；冲突检测：矛盾识别与来源追溯；冲突解决：优先级/证据度/时间衰减等）；Memory Scaling Law（规模维度：数据量/记忆量/模型量/时间维度；扩展规律；最优配置：收益-成本权衡）；参数记忆与外部记忆结合（参数记忆（内隐）+ 外部记忆（外显）分工协同；读写协同与知识迁移；一致性维护）；记忆评估体系（维度：准确性/完整性/时效性/相关性；方法：离线/在线/A-B 测试；指标：检索效果/任务收益/成本效益）；记忆检索与路由（关键词/向量/混合检索；任务感知/意图识别/记忆选择；重排与融合）；记忆遗忘机制（基于时间/重要性/访问频率；软删除/归档/压缩/彻底删除；新旧知识平衡与性能影响）；记忆安全与隐私（权限管理与最小化原则；脱敏/加密/差分隐私；操作日志与可追溯性）。闭环：交互输入 → 写入/更新 → 结构化记忆 → 检索/路由 → 行动输出 → 长期反馈。

## 相关页面

- [[agent-memory-system]] — Agent 长短期记忆系统基础概念
- [[mem0]] — 跨会话用户记忆层（对比：Mem0 在 LoCoMo 上 62.9% vs LycheeMemory 89.2%）
- [[oppo-multimodal-agent-memory]] — OPPO 多模态 Agent 流式记忆与端侧记忆实践
- [[evomembench]] — EvoMemBench 记忆评测基准（评测演进从"记住"到"持续学习"）
- [[agent-evaluation-framework]] — AI Agent 测评框架
- [[agent-hook-governance]] — Agent Hook 治理（插件 Hook 生命周期管理）
- [[business-cognition-system]] — 可演进的业务认知系统（知识沉淀与动态加载）
